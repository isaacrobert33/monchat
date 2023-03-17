<script>
    import { each } from "svelte/internal";
    import { onMount } from "svelte";
    import ChatCard from "./ChatCard.svelte";
    import { createEventDispatcher } from "svelte";
    import { v4 as uuidv4 } from "uuid";

    var host = "http://127.0.0.1:8000";
    const dispatch = createEventDispatcher();

    export let user_data;
    export let chat_recipient_data;
    export let chat_profile_status;
    export let conversation_list = [];
    export let type;

    function deleteText(index) {
        console.log("Deleting text...");
        conversation_list = conversation_list.filter((e, i) => i !== index);
    }

    function generateID(prefix) {
        return `${prefix}_${uuidv4()}`;
    }

    const SingleChatSetup = (msg_input) => {
        let socket_url = `ws://127.0.0.1:8000/ws/chat/${chat_recipient_data.user_id}/`;
        var chat_socket = new WebSocket(socket_url);

        const date = new Date();
        let msg_data = {
            msg_id: generateID("chat"),
            recipient_data: {
                user_id: chat_recipient_data.user_id,
                user_name: chat_recipient_data.user_name,
                user_icon: chat_recipient_data.user_icon,
                first_name: chat_recipient_data.first_name,
                last_name: chat_recipient_data.last_name,
            },
            sender_data: {
                user_id: user_data.user_id,
                user_name: user_data.user_name,
                user_icon: user_data.user_icon,
                first_name: user_data.first_name,
                last_name: user_data.last_name,
            },
            msg_body: msg_input.value,
            msg_sender: user_data.user_name,
            msg_recipient: chat_recipient_data.user_name,
            msg_status: "UD",
        };

        chat_socket.onopen = (e) => {
            msg_data.msg_time = date.toISOString();
            msg_data.direction = "inbound";
            chat_socket.send(JSON.stringify(msg_data));
            chat_socket.close();
        };

        let temp_msg_data = { ...msg_data };
        temp_msg_data.msg_time = `${date
            .getHours()
            .toString()
            .padStart(2, "0")}:${date
            .getMinutes()
            .toString()
            .padStart(2, "0")}`;

        temp_msg_data.direction = "outbound";

        if (chat_profile_status == "online") {
            temp_msg_data.msg_status = "DV";
        }

        dispatch("newmessagesent", temp_msg_data);
        conversation_list = [...conversation_list, temp_msg_data];
        msg_input.value = "";
    };

    onMount(() => {
        let chatlist_node = document.getElementById("chat_con");
        chatlist_node.scrollTop =
            document.getElementById("conv_list").scrollHeight;

        let msg_input = document.getElementById("msg_input");
        msg_input.addEventListener("keydown", function (e) {
            if (e.key === "Enter") {
                if (msg_input.value) {
                    if (type == "single_chat") {
                        SingleChatSetup(msg_input);
                    } else {
                        //
                    }
                }
            }
        });

        var last_msg = conversation_list[conversation_list.length - 1];

        if (
            (last_msg && last_msg.msg_status == "UD") ||
            (last_msg && last_msg.msg_status == "DV")
        ) {
            var msg_socket = new WebSocket(
                `ws://127.0.0.1:8000/ws/read_reciept/${last_msg.msg_id}/`
            );

            msg_socket.onopen = (e) => {
                console.log("SOCKET OPENED FOR READ RECEIPT");

                if (last_msg.direction == "outbound") {
                    console.log("Listening for read receipts...");
                    msg_socket.onmessage = (event) => {
                        const data = JSON.parse(event.data);
                        console.log("Message status changed", data);
                        conversation_list.forEach((chat) => {
                            if (chat.msg_id === data.msg_id) {
                                conversation_list[
                                    conversation_list.indexOf(chat)
                                ].msg_status = data.msg_status;
                            }
                        });
                    };
                } else {
                    if (
                        last_msg.msg_status !== "RD" &&
                        last_msg.direction == "inbound"
                    ) {
                        let dt = new Date();
                        console.log("sent read receipt");
                        dispatch("messageread", last_msg);
                        msg_socket.send(
                            JSON.stringify({
                                msg_id: last_msg.msg_id,
                                msg_status: "RD",
                                read_time: dt.toISOString(),
                            })
                        );
                        msg_socket.close();
                    }
                }
            };
        }
    });
</script>

<div class="chats-container" id="chat_con">
    <div class="right-topbar">
        <img
            class="chat-profile-img"
            src={`${host}/media/${chat_recipient_data.user_icon}/`}
            alt={"profile"}
        />
        <div class="chat-profile-info">
            <p class="chat-profile-name">
                {`${chat_recipient_data.first_name} ${chat_recipient_data.last_name}`}
            </p>
            <p class="chat-profile-status">{chat_profile_status}</p>
        </div>
        <div class="chat-widget">
            <span>
                <svg
                    viewBox="0 0 24 24"
                    height="24"
                    width="24"
                    preserveAspectRatio="xMidYMid meet"
                    class="chat-search-icon"
                    version="1.1"
                    x="0px"
                    y="0px"
                    enable-background="new 0 0 24 24"
                    xml:space="preserve"
                >
                    <path
                        fill="currentColor"
                        d="M15.9,14.3H15L14.7,14c1-1.1,1.6-2.7,1.6-4.3c0-3.7-3-6.7-6.7-6.7S3,6,3,9.7 s3,6.7,6.7,6.7c1.6,0,3.2-0.6,4.3-1.6l0.3,0.3v0.8l5.1,5.1l1.5-1.5L15.9,14.3z M9.7,14.3c-2.6,0-4.6-2.1-4.6-4.6s2.1-4.6,4.6-4.6 s4.6,2.1,4.6,4.6S12.3,14.3,9.7,14.3z"
                    />
                </svg>
            </span>
            <span>
                <svg
                    viewBox="0 0 24 24"
                    height="24"
                    width="24"
                    preserveAspectRatio="xMidYMid meet"
                    class="chat-menu"
                    version="1.1"
                    x="0px"
                    y="0px"
                    enable-background="new 0 0 24 24"
                    xml:space="preserve"
                >
                    <path
                        fill="currentColor"
                        d="M12,7c1.104,0,2-0.896,2-2c0-1.105-0.895-2-2-2c-1.104,0-2,0.894-2,2 C10,6.105,10.895,7,12,7z M12,9c-1.104,0-2,0.894-2,2c0,1.104,0.895,2,2,2c1.104,0,2-0.896,2-2C13.999,9.895,13.104,9,12,9z M12,15 c-1.104,0-2,0.894-2,2c0,1.104,0.895,2,2,2c1.104,0,2-0.896,2-2C13.999,15.894,13.104,15,12,15z"
                    />
                </svg>
            </span>
        </div>
    </div>

    <!-- Chats Start -->

    <div class="conversation-list" id="conv_list">
        {#each conversation_list as conv, index}
            <ChatCard {...conv} bind:msg_status={conv.msg_status} />
        {/each}
    </div>

    <!-- Chats End -->

    <div class="footer">
        <span class="attachment-btn" title="attachment">
            <svg
                viewBox="0 0 24 24"
                height="24"
                width="24"
                preserveAspectRatio="xMidYMid meet"
                class="msg-attachment"
                version="1.1"
                x="0px"
                y="0px"
                enable-background="new 0 0 24 24"
                xml:space="preserve"
            >
                <path
                    fill="currentColor"
                    d="M1.816,15.556v0.002c0,1.502,0.584,2.912,1.646,3.972s2.472,1.647,3.974,1.647 c1.501,0,2.91-0.584,3.972-1.645l9.547-9.548c0.769-0.768,1.147-1.767,1.058-2.817c-0.079-0.968-0.548-1.927-1.319-2.698 c-1.594-1.592-4.068-1.711-5.517-0.262l-7.916,7.915c-0.881,0.881-0.792,2.25,0.214,3.261c0.959,0.958,2.423,1.053,3.263,0.215 c0,0,3.817-3.818,5.511-5.512c0.28-0.28,0.267-0.722,0.053-0.936c-0.08-0.08-0.164-0.164-0.244-0.244 c-0.191-0.191-0.567-0.349-0.957,0.04c-1.699,1.699-5.506,5.506-5.506,5.506c-0.18,0.18-0.635,0.127-0.976-0.214 c-0.098-0.097-0.576-0.613-0.213-0.973l7.915-7.917c0.818-0.817,2.267-0.699,3.23,0.262c0.5,0.501,0.802,1.1,0.849,1.685 c0.051,0.573-0.156,1.111-0.589,1.543l-9.547,9.549c-0.756,0.757-1.761,1.171-2.829,1.171c-1.07,0-2.074-0.417-2.83-1.173 c-0.755-0.755-1.172-1.759-1.172-2.828l0,0c0-1.071,0.415-2.076,1.172-2.83c0,0,5.322-5.324,7.209-7.211 c0.157-0.157,0.264-0.579,0.028-0.814c-0.137-0.137-0.21-0.21-0.342-0.342c-0.2-0.2-0.553-0.263-0.834,0.018 c-1.895,1.895-7.205,7.207-7.205,7.207C2.4,12.645,1.816,14.056,1.816,15.556z"
                />
            </svg>
        </span>
        <input
            type="text"
            class="msg-input"
            id="msg_input"
            placeholder="Enter a message..."
        />
        <span>
            <svg
                viewBox="0 0 24 24"
                height="24"
                width="24"
                preserveAspectRatio="xMidYMid meet"
                class="msg-mic"
                version="1.1"
                x="0px"
                y="0px"
                enable-background="new 0 0 24 24"
                xml:space="preserve"
            >
                <path
                    fill="currentColor"
                    d="M11.999,14.942c2.001,0,3.531-1.53,3.531-3.531V4.35c0-2.001-1.53-3.531-3.531-3.531 S8.469,2.35,8.469,4.35v7.061C8.469,13.412,9.999,14.942,11.999,14.942z M18.237,11.412c0,3.531-2.942,6.002-6.237,6.002 s-6.237-2.471-6.237-6.002H3.761c0,4.001,3.178,7.297,7.061,7.885v3.884h2.354v-3.884c3.884-0.588,7.061-3.884,7.061-7.885 L18.237,11.412z"
                />
            </svg>
        </span>
    </div>
</div>

<style>
    .chats-container {
        height: 100%;
        width: 70%;
        overflow: auto;
        scrollbar-width: thin;
        scrollbar-color: rgba(240, 255, 255, 0.2) transparent;
        margin-left: 30%;
        background-color: #111b21;
    }

    .conversation-list {
        overflow: auto;
        overflow-x: hidden;
        padding: 60px 0;
    }

    .right-topbar {
        position: fixed;
        top: 0;
        width: 100%;
        height: 50px;
        border-left: 0.5px solid rgba(255, 255, 255, 0.5);
        background-color: #202c33;
        padding: 0;
        z-index: 3;
        overflow: hidden;
    }

    .chat-profile-img {
        width: 40px;
        height: 40px;
        margin: 5px;
        margin-left: 10px;
        border-radius: 50%;
        display: inline-block;
    }

    .chat-widget {
        display: inline-block;
        position: absolute;
        margin: 10px;
        right: 30%;
        color: azure;
        cursor: pointer;
    }

    .chat-widget svg {
        margin: 0 10px;
    }

    .chat-profile-info {
        color: azure;
        width: 15%;
        margin: 5px 10px;
        position: absolute;
        display: inline-block;
    }

    .chat-profile-name {
        margin: 2px;
    }

    .chat-profile-status {
        margin: 2px;
        color: rgba(240, 255, 255, 0.5);
        font-size: 13px;
    }

    .footer {
        bottom: 0;
        position: fixed;
        width: 100%;
        height: 50px;
        border-left: 0.5px solid rgba(255, 255, 255, 0.5);
        background-color: #202c33;
        z-index: 3;
    }

    .msg-attachment {
        color: rgba(240, 255, 255, 0.5);
        cursor: pointer;
        margin-top: 13px;
        margin-left: 10px;
        position: absolute;
    }

    .msg-input {
        width: 63%;
        height: 80%;
        color: azure;
        background-color: #2a3942;
        margin-top: 6px;
        margin-left: 50px;
        border: none;
        border-radius: 8px;
        padding: 20px;
        outline: none;
    }

    .msg-mic {
        color: rgba(240, 255, 255, 0.5);
        cursor: pointer;
        margin-top: 13px;
        margin-left: 10px;
        position: absolute;
    }
</style>
