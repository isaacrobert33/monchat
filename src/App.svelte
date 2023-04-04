<script>
  import axios from "axios";
  import { onMount } from "svelte";
  import { format } from "timeago.js";
  import ChatContainer from "./components/ChatContainer.svelte";
  import ListContainer from "./components/ListContainer.svelte";
  import Login from "./components/Login.svelte";

  export let name;

  const host = "http://monchat.pythonanywhere.com/monchat";

  let user_id = window.localStorage.getItem("monchat_user_id");
  var user_data = {};
  var groups = [];
  var chatList = [];
  let currentChatStatus = false;

  var groupSockets = {};
  const initializeGroupSocket = (groups) => {
    for (let i = 0; i < groups.length; i++) {
      let socket_url = `ws://127.0.0.1:8000/ws/group/${groups[i]}/`;
      let group_socket = new WebSocket(socket_url);

      group_socket.onopen = (e) => {
        console.log("GROUP SOCKET OPENED FOR ", groups[i]);
      };

      group_socket.onclose = (e) => {
        console.log("GROUP CLOSED FOR ", groups[i]);
      };

      group_socket.onmessage = (event) => {
        let data = JSON.parse(event.data);
        event = { detail: data };

        if (event.detail.msg_sender !== user_data.user_name) {
          handleNewMsg(event, "inbound");
        }
      };

      groupSockets[groups[i]] = group_socket;
    }
  };

  let typing_socket;

  const initialize_socket = () => {
    ////// Initialize self socket
    let socket_url = `ws://127.0.0.1:8000/ws/chat/${user_data.user_id}/`;
    let chat_socket = new WebSocket(socket_url);

    chat_socket.onopen = (e) => {
      console.log("SOCKET OPEN FOR INCOMING MESSAGE");
    };

    chat_socket.onclose = (e) => {
      console.log("YOUR SOCKET WAS CLOSED");
    };

    chat_socket.onmessage = (event) => {
      let data = JSON.parse(event.data);
      event = { detail: data };
      handleNewMsg(event, "inbound");
    };

    ///// Initializing online status socket
    let online_socket_url = `ws://127.0.0.1:8000/ws/online/`;
    let online_socket = new WebSocket(online_socket_url);

    online_socket.onopen = function (e) {
      console.log("CONNECTED TO ONLINE CONSUMER");
      online_socket.send(
        JSON.stringify({
          user_name: user_data.user_name,
          type: "open",
        })
      );
    };

    window.addEventListener("beforeunload", function (e) {
      alert("Unload");
      let dt = new Date();

      online_socket.send(
        JSON.stringify({
          user_name: user_data.user_name,
          type: "offline",
          time: dt.toISOString(),
        })
      );
    });

    online_socket.onclose = function (e) {
      console.log("DISCONNECTED FROM ONLINE CONSUMER");
    };

    online_socket.onmessage = function (e) {
      let data = JSON.parse(e.data);

      if (openedChatData && openedChatData.user_name) {
        if (data.user_name == openedChatData.user_name) {
          if (data.online_status == true) {
            currentChatStatus = "online";
          } else {
            let dt = new Date(data.time);
            let dt_str = `Last seen ${dt
              .getHours()
              .toString()
              .padStart(2, "0")}:${dt
              .getMinutes()
              .toString()
              .padStart(2, "0")}`;
            currentChatStatus = dt_str;
          }
        }
      }
    };

    let typing_url = `ws://127.0.0.1:8000/ws/typing/`;
    typing_socket = new WebSocket(typing_url);

    typing_socket.onclose = function (e) {
      console.log("DISCONNECTED FROM TYPING CONSUMER");
    };

    typing_socket.onopen = function (e) {
      console.log("CONNECTED TO TYPING CONSUMER");
    };

    typing_socket.onmessage = function (e) {
      let data = JSON.parse(e.data);
      setTyping(data);
    };
  };

  const setTyping = (event) => {
    let i;
    let tempChatList = chatList;

    for (i = 0; i < tempChatList.length; i++) {
      let recipient =
        tempChatList[i].direction == "outbound"
          ? tempChatList[i].msg_recipient
          : tempChatList[i].msg_sender;

      if (event.user_name === recipient.user_name) {
        tempChatList[i].typing = event.typing;
      }
    }

    chatList = tempChatList;
  };

  const sendTyping = (event) => {
    if (typing_socket) {
      typing_socket.send(event.detail);
    }
  };

  const fetchCL = async () => {
    await axios
      .get(`${host}/user/${user_id}/`, {
        headers: {
          "Content-Type": "application/json",
        },
      })
      .then((response) => {
        user_data = response.data.data;
      })
      .catch((err) => {
        window.localStorage.setItem("monchat_user_id", "");
        window.location.reload();
      });

    await axios
      .get(`${host}/latest_chats/${user_id}/`, {
        headers: {
          "Content-Type": "application/json",
        },
      })
      .then((response) => {
        chatList = response.data.data;
        groups = response.data.groups;

        initialize_socket();
        initializeGroupSocket(response.data.groups);
      });
  };

  if (user_id) {
    fetchCL();
  }

  const handleLogin = (event) => {
    user_id = event.detail.user_id;
    user_data = event.detail;
    // window.localStorage.setItem("monchat_user_id", user_id);
    fetchCL();
  };

  const fetchRecipientData = async (recipient) => {
    await axios
      .get(`${host}/user/${recipient.user_id}/`, {
        headers: {
          "Content-Type": "application/json",
        },
      })
      .then((response) => {
        openedChatData = response.data.data;
        let last_seen = `Last seen ${format(response.data.data.last_seen)}`;
        currentChatStatus =
          openedChatData.online_status == false ? last_seen : "online";

        fetchChats(openedChatData);
      })
      .catch((err) => {
        alert("Error fetching recipient data");
        console.log(err);
      });
  };

  const fetchChats = async (openedChatData) => {
    await axios
      .get(
        chatType == "single_chat"
          ? `${host}/chats/${user_data.user_name}/${openedChatData.user_name}/`
          : `${host}/group_chats/${openedChatData.group_id}/${user_data.user_name}/`,
        {
          headers: {
            "Content-Type": "application/json",
          },
        }
      )
      .then((response) => {
        conversations = response.data.data;
        let last_msg = conversations[conversations.length - 1];

        if (conversations.length > 0) {
          execLastMsgAction(last_msg);
        }
      });
  };

  const execLastMsgAction = (last_msg) => {
    let surl = `ws://127.0.0.1:8000/ws/read_reciept/${last_msg.msg_id}/`;
    let last_msg_socket = new WebSocket(surl);

    last_msg_socket.onclose = (e) => {
      console.log("READ RECEIPT SOCKET CLOSED");
    };

    if (last_msg.direction == "inbound") {
      if (last_msg.msg_status == "UD" || last_msg.msg_status == "DV") {
        last_msg_socket.onopen = (e) => {
          let dt = new Date();

          handleMsgRead(last_msg, last_msg.direction, chatType);

          last_msg_socket.send(
            JSON.stringify({
              msg_id: last_msg.msg_id,
              msg_sender: openedChatData.user_name,
              msg_recipient: user_data.user_name,
              msg_status: "RD",
              read_time: dt.toISOString(),
            })
          );
          last_msg_socket.close();
        };
      } else {
        last_msg_socket.close();
      }
    } else {
      if (last_msg.msg_status == "RD") {
        last_msg_socket.close();
      } else {
        last_msg_socket.onmessage = (event) => {
          let data = JSON.parse(event.data);
          handleMsgRead(data, last_msg.direction);
        };
      }
    }
  };

  let conversations = [];
  let openedChatData = null;
  let activeFailed;
  var chatType;
  var activeChat;

  const openChat = async (event) => {
    chatType = event.detail.type;

    // setting colors for active chats
    if (activeChat) {
      document.getElementById(activeChat).style.backgroundColor = "#111b21";
    }
    activeChat = `sidebar-${
      chatType == "single_chat"
        ? event.detail.msg_id
        : event.detail.group_data.group_id
    }`;

    if (document.getElementById(activeChat)) {
      document.getElementById(activeChat).style.backgroundColor = "#2a3942";
    }

    let recipient =
      event.detail.direction == "outbound"
        ? event.detail.msg_recipient
        : event.detail.msg_sender;

    if (chatType == "single_chat") {
      fetchRecipientData(recipient);
    } else {
      openedChatData = event.detail.group_data;
      fetchChats(openedChatData);
    }
  };

  const updateSideBarChats = (new_chat) => {
    let updated = false;

    chatList.forEach((chat) => {
      if (chat.type == "single_chat") {
        if (
          (chat.msg_recipient.user_name == new_chat.msg_sender &&
            chat.msg_sender.user_name == new_chat.msg_recipient) ||
          (chat.msg_recipient.user_name === new_chat.msg_recipient &&
            chat.msg_sender.user_name === new_chat.msg_sender)
        ) {
          let index = chatList.indexOf(chat);
          let actualChat = chatList[index];
          let temp_chatList = chatList;

          let date = new Date(new_chat.msg_time);
          let fTime =
            new_chat.direction == "outbound"
              ? new_chat.msg_time
              : `${date.getHours().toString().padStart(2, "0")}:${date
                  .getMinutes()
                  .toString()
                  .padStart(2, "0")}`;

          let updated_data = {
            direction: new_chat.direction,
            unread_count:
              new_chat.direction == "inbound"
                ? actualChat.unread_count + 1
                : actualChat.unread_count,
            msg_time: fTime,
            msg_body: new_chat.msg_body,
            msg_sender:
              new_chat.direction == "outbound" ? user_data : openedChatData,
            msg_recipient:
              new_chat.direction == "inbound" ? user_data : openedChatData,
            msg_status: new_chat.msg_status,
            msg_id: new_chat.msg_id,
            msg_timeago: format(new_chat.msg_time),
            type: actualChat.type,
          };

          updated = true;
          temp_chatList.splice(index, 1);
          temp_chatList.unshift(updated_data);
          chatList = temp_chatList;
        }
      } else {
        if (chat.group_id == new_chat.group_id) {
          let index = chatList.indexOf(chat);
          let actualChat = chatList[index];
          let temp_chatList = chatList;

          let date = new Date(new_chat.msg_time);
          let fTime =
            new_chat.direction == "outbound"
              ? new_chat.msg_time
              : `${date.getHours().toString().padStart(2, "0")}:${date
                  .getMinutes()
                  .toString()
                  .padStart(2, "0")}`;

          let updated_data = { ...actualChat };
          updated_data.msg_time = fTime;
          updated_data.direction = new_chat.direction;
          updated_data.type = "group_chat";
          updated_data.msg_timeago = format(new_chat.msg_time);
          updated_data.msg_status = new_chat.msg_status;
          updated_data.msg_body = new_chat.msg_body;
          updated_data.msg_sender = new_chat.sender_data;
          updated_data.unread_count =
            new_chat.direction == "inbound"
              ? actualChat.unread_count + 1
              : actualChat.unread_count;

          updated = true;

          updated = true;
          temp_chatList.splice(index, 1);
          temp_chatList.unshift(updated_data);
          chatList = temp_chatList;
        }
      }
    });

    if (!updated) {
      let fTime = format(new_chat.msg_time);
      const chat_data =
        new_chat.type == "single_chat"
          ? {
              msg_recipient: new_chat.recipient_data,
              msg_sender:
                user_data.user_name == new_chat.msg_sender
                  ? user_data
                  : new_chat.sender_data,
              msg_body: new_chat.msg_body,
              msg_time:
                new_chat.direction == "inbound" ? fTime : new_chat.msg_time,
              direction: new_chat.direction,
              unread_count: new_chat.direction == "inbound" ? 1 : 0,
              msg_status: new_chat.msg_status,
              type: new_chat.type,
            }
          : {
              group_data: new_chat.group_data,
              msg_sender:
                user_data.user_name == new_chat.msg_sender
                  ? user_data
                  : new_chat.sender_data,
              msg_body: new_chat.msg_body,
              msg_time:
                new_chat.direction == "inbound" ? fTime : new_chat.msg_time,
              direction: new_chat.direction,
              unread_count: new_chat.direction == "inbound" ? 1 : 0,
              msg_status: new_chat.msg_status,
              type: new_chat.type,
            };
      console.log("cd", chat_data);
      let tcl = chatList;
      tcl.unshift(chat_data);

      chatList = tcl;
    }
  };

  var msg_socket;
  const handleNewMsg = (event, direction = "outbound") => {
    console.log(event.detail);
    updateSideBarChats(event.detail);
    let conv_data = event.detail;

    if (msg_socket) {
      msg_socket.close();
    }

    msg_socket = new WebSocket(
      `ws://127.0.0.1:8000/ws/read_reciept/${conv_data.msg_id}/`
    );
    msg_socket.onopen = (e) => {
      console.log("SOCKET OPENED FOR READ RECEIPT -", conv_data.msg_id);
    };

    msg_socket.onclose = (e) => {
      console.log("READ RECEIPT SOCKET CLOSED -", conv_data.msg_id);
    };

    if (direction == "inbound") {
      let date = new Date(conv_data.msg_time);

      conv_data.msg_time = `${date
        .getHours()
        .toString()
        .padStart(2, "0")}:${date.getMinutes().toString().padStart(2, "0")}`;
      conv_data.direction = "inbound";

      conv_data.msg_sender = conv_data.sender_data;

      conversations = [...conversations, conv_data];

      let chatlist_node = document.getElementById("chat_con");

      if (chatlist_node) {
        chatlist_node.scrollTop =
          document.getElementById("conv_list").scrollHeight;
      }

      var type;

      const sendReadReceipt = () => {
        let dt = new Date();

        if (type == "single_chat") {
          handleMsgRead(conv_data, direction);
          msg_socket.send(
            JSON.stringify({
              msg_id: conv_data.msg_id,
              msg_status: "RD",
              read_time: dt.toISOString(),
            })
          );
          document.removeEventListener("mousemove", sendReadReceipt);
        } else {
          handleMsgRead(conv_data, direction, "group");
          msg_socket.send(
            JSON.stringify({
              msg_id: conv_data.msg_id,
              msg_status: "RD",
              read_time: dt.toISOString(),
              read_by: user_data.user_name,
            })
          );
          document.removeEventListener("mousemove", sendReadReceipt);
        }
      };

      if (openedChatData) {
        if (
          chatType == "single_chat" &&
          openedChatData.user_name == conv_data.msg_sender
        ) {
          type = "single_chat";
          document.addEventListener("mousemove", sendReadReceipt);
        } else if (openedChatData.group_id == conv_data.group_data.group_id) {
          type = "group_chat";

          document.addEventListener("mousemove", sendReadReceipt);
        }
      }
    } else {
      msg_socket.onmessage = (event) => {
        const data = JSON.parse(event.data);
        handleMsgRead(data, direction);
      };
    }
  };

  const handleMsgRead = (event, direction, type = "single_chat") => {
    if (direction == "outbound") {
      let chat_index = conversations.length - 1;

      for (let i = 0; i < conversations.length; i++) {
        if (conversations[i].msg_id == event.msg_id) {
          chat_index = i;
          break;
        }
      }

      let temp = conversations;
      for (let j = 0; j < temp.length; j++) {
        if (temp[j].type == "single_chat") {
          temp[j].msg_status = "RD";
          if (j == chat_index) {
            break;
          }
        } else {
          temp[j].read_by = [...temp[j].read_by, user_data.user_name];
        }
      }

      conversations = temp;
    } else {
      chatList.forEach((chat) => {
        if (chat.type == "single_chat") {
          if (
            (chat.msg_recipient.user_name == event.msg_sender &&
              chat.msg_sender.user_name == event.msg_recipient) ||
            (chat.msg_recipient.user_name === event.msg_recipient &&
              chat.msg_sender.user_name === event.msg_sender)
          ) {
            let index = chatList.indexOf(chat);
            let c = chatList[index];
            let updated = {
              ...c,
              unread_count: 0,
            };

            chatList[index] = updated;
          }
        } else {
          if (chat.group_data.group_id == event.group_id) {
            let index = chatList.indexOf(chat);
            let c = chatList[index];
            let updated = {
              ...c,
              unread_count: 0,
            };

            chatList[index] = updated;
          }
        }
      });
    }
  };

  const handleNewChat = async (event) => {
    openedChatData = event.detail;

    await axios
      .get(`${host}/user_status/${openedChatData.user_id}/`, {
        headers: {
          "Content-Type": "application/json",
        },
      })
      .then((response) => {
        let last_seen = `Last seen ${format(response.data.data.last_seen)}`;
        currentChatStatus =
          response.data.data.online_status == false ? last_seen : "online";
      });

    await axios
      .get(
        `${host}/chats/${user_data.user_name}/${openedChatData.user_name}/`,
        {
          headers: {
            "Content-Type": "application/json",
          },
        }
      )
      .then((response) => {
        conversations = response.data.data;
      });
  };

  const handleNewGroup = (event) => {
    let groupData = event.detail;
    let newChatList = chatList;
    newChatList.unshift(groupData);
    chatList = newChatList;
    openChat(event);
  };
</script>

<div class="App {name}">
  {#if !user_id}
    <Login on:login={handleLogin} />
  {/if}
  {#if user_data.user_name}
    <ListContainer
      bind:chatList
      {user_data}
      on:chatclick={openChat}
      on:newchat={handleNewChat}
      on:newgroup={handleNewGroup}
    />
  {/if}
  {#if openedChatData}
    <ChatContainer
      bind:chat_recipient_data={openedChatData}
      type={chatType}
      {user_data}
      bind:conversation_list={conversations}
      chat_profile_status={currentChatStatus}
      on:newmessagesent={handleNewMsg}
      on:messageread={handleMsgRead}
      on:typing={sendTyping}
    />
  {/if}
</div>

<style>
  .App {
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
    background-color: #222e35;
  }
  :global(body) {
    margin: 0;
    padding: 0;
  }
</style>
