<script>
	import axios from "axios";
	import ChatContainer from "./components/ChatContainer.svelte";
	import ListContainer from "./components/ListContainer.svelte";
	import Login from "./components/Login.svelte";

	export let name;

	const host = "http://127.0.0.1:8000/monchat";

	let user_id = window.localStorage.getItem("monchat_user_id");
	let user_data = {};
	let chat_list = [];
	let currentChatStatus = false;

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
			console.log("Incoming", event);
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

			if (openedChatData.user_name) {
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
				initialize_socket();
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
				chat_list = response.data.data;
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
				let dt = new Date(openedChatData.last_seen);
				let last_seen = `Last seen ${dt
					.getHours()
					.toString()
					.padStart(2, "0")}:${dt
					.getMinutes()
					.toString()
					.padStart(2, "0")}`;
				currentChatStatus =
					openedChatData.online_status == false
						? last_seen
						: "online";

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
				`${host}/chats/${user_data.user_name}/${openedChatData.user_name}/`,
				{
					headers: {
						"Content-Type": "application/json",
					},
				}
			)
			.then((response) => {
				conversations = response.data.data;
				let last_msg = conversations[conversations.length - 1];
				console.log(conversations.length);
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
					console.log("sent read receipt");
					handleMsgRead(last_msg);

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
				console.log("Waiting for msg read");
				last_msg_socket.onmessage = (event) => {
					console.log("Message read!");
					let data = JSON.parse(event.data);
					handleMsgRead(data, last_msg.direction);
				};
			}
		}
	};

	let conversations = [];
	let openedChatData = {};
	var activeChat;

	const openChat = async (event) => {
		// setting colors for active chats
		if (activeChat) {
			document.getElementById(activeChat).style.backgroundColor =
				"#111b21";
		}
		document.getElementById(
			`sidebar-${event.detail.msg_id}`
		).style.backgroundColor = "#202c33";
		activeChat = `sidebar-${event.detail.msg_id}`;

		let recipient =
			event.detail.direction == "outbound"
				? event.detail.msg_recipient
				: event.detail.msg_sender;

		fetchRecipientData(recipient);
	};

	const updateSideBarChats = (new_chat) => {
		let updated = false;

		chat_list.forEach((chat) => {
			if (
				(chat.msg_recipient.user_name == new_chat.msg_sender &&
					chat.msg_sender.user_name == new_chat.msg_recipient) ||
				(chat.msg_recipient.user_name === new_chat.msg_recipient &&
					chat.msg_sender.user_name === new_chat.msg_sender)
			) {
				let index = chat_list.indexOf(chat);
				let actualChat = chat_list[index];
				let temp_chat_list = chat_list;

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
					msg_sender: actualChat.msg_recipient,
					msg_recipient: actualChat.msg_sender,
					msg_status: new_chat.msg_status,
					msg_id: new_chat.msg_id,
				};
				console.log("Updt", updated_data);
				updated = true;
				temp_chat_list.splice(index, 1);
				temp_chat_list.unshift(updated_data);
				chat_list = temp_chat_list;
			}
		});

		if (!updated) {
			console.log("new_chat", new_chat);

			let date = new Date(new_chat.msg_time);

			let fTime = `${date.getHours().toString().padStart(2, "0")}:${date
				.getMinutes()
				.toString()
				.padStart(2, "0")}`;
			const chat_data = {
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
			};
			console.log("chat_data", chat_data);
			let tcl = chat_list;
			tcl.unshift(chat_data);

			chat_list = tcl;
		}
	};

	var msg_socket;
	const handleNewMsg = (event, direction = "outbound") => {
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
				.padStart(2, "0")}:${date
				.getMinutes()
				.toString()
				.padStart(2, "0")}`;
			conv_data.direction = "inbound";

			conversations = [...conversations, conv_data];

			let chatlist_node = document.getElementById("chat_con");

			if (chatlist_node) {
				chatlist_node.scrollTop =
					document.getElementById("conv_list").scrollHeight;
			}

			const sendReadReceipt = () => {
				let dt = new Date();

				console.log("sent read receipt");
				handleMsgRead(conv_data, direction);
				msg_socket.send(
					JSON.stringify({
						msg_id: conv_data.msg_id,
						msg_status: "RD",
						read_time: dt.toISOString(),
					})
				);
				document.removeEventListener("mousemove", sendReadReceipt);
			};
			console.log("op", openedChatData);
			if (
				openedChatData &&
				openedChatData.user_name == conv_data.msg_sender
			) {
				document.addEventListener("mousemove", sendReadReceipt);
			}
		} else {
			console.log("Listening for read receipt");
			msg_socket.onmessage = (event) => {
				const data = JSON.parse(event.data);
				console.log("Message status changed", data);
				handleMsgRead(data, direction);
			};
		}
	};

	const handleMsgRead = (event, direction) => {
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
				temp[j].msg_status = "RD";
				if (j == chat_index) {
					break;
				}
			}

			conversations = temp;
		} else {
			chat_list.forEach((chat) => {
				if (
					(chat.msg_recipient.user_name == event.msg_sender &&
						chat.msg_sender.user_name == event.msg_recipient) ||
					(chat.msg_recipient.user_name === event.msg_recipient &&
						chat.msg_sender.user_name === event.msg_sender)
				) {
					let index = chat_list.indexOf(chat);
					let c = chat_list[index];
					let updated = {
						...c,
						unread_count: 0,
					};

					chat_list[index] = updated;
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
				let dt = new Date(response.data.data.last_seen);
				let last_seen = `Last seen ${dt
					.getHours()
					.toString()
					.padStart(2, "0")}:${dt
					.getMinutes()
					.toString()
					.padStart(2, "0")}`;
				currentChatStatus =
					response.data.data.online_status == false
						? last_seen
						: "online";
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
</script>

<div class="App {name}">
	{#if !user_id}
		<Login on:login={handleLogin} />
	{/if}
	{#if user_data.user_name}
		<ListContainer
			bind:chat_list
			{user_data}
			on:chatclick={openChat}
			on:newchat={handleNewChat}
		/>
	{/if}
	{#if openedChatData.user_id}
		<ChatContainer
			bind:chat_recipient_data={openedChatData}
			{user_data}
			bind:conversation_list={conversations}
			chat_profile_status={currentChatStatus}
			on:newmessagesent={handleNewMsg}
			on:messageread={handleMsgRead}
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
