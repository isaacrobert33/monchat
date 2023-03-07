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
		await axios.get(`${host}/latest_chats/${user_id}/`, {
			headers: {
				'Content-Type': 'application/json'
			}
		}).then(
			response => {
				console.log(response.data.data);
				chat_list = response.data.data;
			}
		)

		await axios.get(`${host}/user/${user_id}/`, {
			headers: {
				'Content-Type': 'application/json'
			}
		}).then(
			response => {
				user_data = response.data.data;
			}
		)
	}

	if (user_id) {
		fetchCL();
	}

	const handleLogin = (event) => {
		user_id = event.detail.user_id;
		user_data = event.detail;
		window.localStorage.setItem("monchat_user_id", user_id);
		console.log("login in", event.detail)
		fetchCL()
	}

	
	let conversations = [];
	let openedChatData = {};
	const openChat = async (event) => {
		openedChatData = event.detail;
		await axios.get(`${host}/chats/${event.detail.msg_sender.user_name}/${openedChatData.msg_recipient.user_name}/`, {
			headers: {
				'Content-Type': 'application/json'
			}
		}).then(
			response => {
				conversations = response.data.data;
			}
		)
	}

	
</script>

<div class="App">
	{#if !user_id}
		<Login on:login={handleLogin} />
	{/if}
	<ListContainer chat_list={chat_list} profile_img={`./profile_assets/${user_data.user_icon}`} on:chatclick={openChat}/>
	{#if openedChatData.msg_recipient}
		<ChatContainer user_id={user_id} chat_recipient_id={openedChatData.msg_recipient.user_name} chat_profile_name={openedChatData.msg_recipient.user_name} conversation_list={conversations}  />
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