<script>
	import axios from "axios";
	import ChatContainer from "./components/ChatContainer.svelte";
	import ListContainer from "./components/ListContainer.svelte";
	import Login from "./components/Login.svelte";

	const host = "http://127.0.0.1:8000/monchat";

	let user_id = window.localStorage.getItem("monchat_user_id");
	let user_data = {};
	let chat_list = [];
	let currentChatStatus = false;

	const initialize_socket = () => {
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
		window.localStorage.setItem("monchat_user_id", user_id);
		console.log("login in", event.detail);
		fetchCL();
	};

	const sendReadReceipt = (msg_socket, msg_id) => {
		let dt = new Date();
		console.log("sent read receipt");
		msg_socket.send(
			JSON.stringify({
				msg_id: msg_id,
				msg_status: ("RD", "Read"),
				read_time: dt.toISOString(),
			})
		);
		document
			.getElementById("conv_list")
			.removeEventListener("focus", (e) => {
				sendReadReceipt(msg_id);
			});
	};

	let conversations = [];
	let openedChatData = {};
	let socketID = null;
	let latest_msg_data = null;

	const openChat = async (event) => {
		let recipient =
			event.detail.direction == "outbound"
				? event.detail.msg_recipient
				: event.detail.msg_sender;

		latest_msg_data = event.detail;

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
			})
			.catch((err) => {
				alert("Error fetching recipient data");
				console.log(err);
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
				socketID = response.data.socket_id;
			});
	};
</script>

<div class="App">
	{#if !user_id}
		<Login on:login={handleLogin} />
	{/if}
	{#if user_data.user_name}
		<ListContainer
			bind:chat_list
			profile_img={user_data.user_icon}
			user_name={user_data.user_name}
			on:chatclick={openChat}
		/>
	{/if}
	{#if openedChatData.user_id && socketID}
		<ChatContainer
			{latest_msg_data}
			chat_recipient_data={openedChatData}
			{user_data}
			bind:conversation_list={conversations}
			{socketID}
			chat_profile_status={currentChatStatus}
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
