<script>
	import axios from "axios";
	import ChatContainer from "./components/ChatContainer.svelte";
	import ListContainer from "./components/ListContainer.svelte";
	import Login from "./components/Login.svelte";

	const host = "http://127.0.0.1:8000/monchat";

	let user_id = window.localStorage.getItem("monchat_user_id");
	let user_data = {};
	let chat_list = [];

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

	let conversations = [];
	let openedChatData = {};
	let socketID = null;
	const openChat = async (event) => {
		openedChatData = event.detail;
		await axios
			.get(
				`${host}/chats/${user_data.user_name}/${
					openedChatData.direction == "outbound"
						? openedChatData.msg_recipient.user_name
						: openedChatData.msg_sender.user_name
				}/`,
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
			{chat_list}
			profile_img={user_data.user_icon}
			user_name={user_data.user_name}
			on:chatclick={openChat}
		/>
	{/if}
	{#if openedChatData.msg_recipient && socketID}
		<ChatContainer
			{openedChatData}
			{user_data}
			conversation_list={conversations}
			{socketID}
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
