<script>
    import { each } from "svelte/internal";
import ListItem from "./ListItem.svelte";
    let profile_img = './assets/user.svg';

    export let chat_list = [
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        {chat_title: "Mi Padre", chat_msg_prev: "Salut Robert, Comment tu vas?", chat_icon: "./assets/user.svg"},
        ]
</script>

<div class="list-container">
    <div class="side-topbar">
        <img class="profile-img" src={profile_img} alt={"profile"} />
    </div>
    <input type="text" class="chat-search" placeholder="Search or start a new chat"/>
    <svg class="filter" viewBox="0 0 24 24" height="22" width="22" preserveAspectRatio="xMidYMid meet" version="1.1" x="0px" y="0px" enable-background="new 0 0 24 24" xml:space="preserve">
        <path fill="currentColor" d="M10,18.1h4v-2h-4V18.1z M3,6.1v2h18v-2H3z M6,13.1h12v-2H6V13.1z"></path>
    </svg>

    {#each chat_list as chat}
        <ListItem {...chat} />
    {/each}
    <!-- <ListItem chat_title={"Mi Padre"} chat_msg_prev={"Salut Robert, Comment tu vas?"} chat_icon={'./assets/user.svg'} /> -->
</div>

<style>
    .list-container {
        display: inline-block;
        width: 30%;
        margin: 0;
        height: 100%;
        padding: 10px 0;
        background-color: #111b21;
        margin-top: 50px;
        position: fixed;
        overflow: auto;
        border-right: 0.5px solid rgba(255, 255, 255, 0.5);
    }

    .side-topbar {
        border-right: 0.5px solid rgba(255, 255, 255, 0.5);
        background-color: #202c33;
        width: 30%;
        height: 50px;
        margin: 0;
        top: 0;
        right: 0;
        left: 0;
        z-index: 2;
        position: fixed;
    }
    .profile-img {
        width: 40px;
        height: 40px;
        margin: 5px;
        margin-left: 10px;
    }

    .chat-search {
        width: 85%;
        margin-left: 10px;
        background-color: #202c33;
        border-radius: 9px;
        border: none;
        color: #f0ffff;
        outline: none;
    }

    .filter {
        color: rgba(240, 255, 255, 0.5);
        margin-left: 10px;
        cursor: pointer;
    }
</style>