<script>
    import { each } from "svelte/internal";
    import { createEventDispatcher } from "svelte";
    import { onMount } from "svelte";
    import axios from "axios";
    import Profile from "./Profile.svelte";
    import Chats from "./Chats.svelte";
    import Users from "./Users.svelte";
    import NewGroupMembers from "./NewGroupMembers.svelte";
    import NewGroup from "./NewGroup.svelte";

    export let chat_list;
    export let user_data;

    const dispatch = createEventDispatcher();
    const host = "http://127.0.0.1:8000";

    let originalChatList = [];
    $: if (!originalChatList.length > 0) {
        originalChatList = chat_list;
    }

    function handleChatClick(event) {
        dispatch("chatclick", {
            ...event.detail,
        });
    }

    var sidebar = "chats";
    var mutated = false;

    var newGrpMembers = [];

    const saveProfileChanges = async () => {
        let payload = {
            fname: document.getElementById("fname").value,
            lname: document.getElementById("lname").value,
            user_bio: document.getElementById("bio").value,
        };
        await axios
            .put(`${host}/monchat/user/${user_data.user_id}/`, payload, {
                headers: { "Content-Type": "application/json" },
            })
            .then((response) => {
                user_data = response.data.data;
            })
            .catch((err) => {
                alert(err);
            });
    };
</script>

<div class="list-container">
    {#if sidebar === "profile"}
        <Profile
            bind:user_data
            on:onback={(e) => {
                if (mutated) {
                    saveProfileChanges();
                }
                sidebar = "chats";
            }}
            on:mutated={(e) => {
                mutated = true;
            }}
        />
    {:else if sidebar == "chats"}
        <Chats
            bind:user_data
            bind:chat_list
            bind:originalChatList
            on:chatclick={handleChatClick}
            on:profile={(e) => {
                sidebar = "profile";
            }}
            on:users={(e) => {
                sidebar = "users";
            }}
            on:newgroup={(e) => {
                sidebar = "new_group_choice";
            }}
        />
    {:else if sidebar == "users"}
        <Users
            bind:user_data
            on:chats={(e) => {
                sidebar = "chats";
            }}
            on:newchat={(e) => {
                dispatch("newchat", e.detail);
            }}
        />
    {:else if sidebar == "new_group_choice"}
        <NewGroupMembers
            {user_data}
            on:newgroup={(e) => {
                sidebar = "new_group";
                newGrpMembers = e.detail.members;
            }}
            on:back={(e) => {
                sidebar = "chats";
            }}
        />
    {:else if sidebar == "new_group"}
        <NewGroup
            members={newGrpMembers}
            {user_data}
            on:back={(e) => {
                sidebar = "chats";
            }}
            on:groupcreate={(e) => {
                console.log("new group", e.detail);
            }}
        />
    {/if}
</div>

<style>
    .list-container {
        display: inline-block;
        width: 30%;
        margin: 0;
        height: 100%;
        padding: 10px 0;
        padding-bottom: 60px;
        background-color: #111b21;
        margin-top: 50px;
        position: fixed;
        overflow-y: auto;
        scrollbar-width: thin;
        scrollbar-color: rgba(240, 255, 255, 0.2) transparent;
        border-right: 0.5px solid rgba(255, 255, 255, 0.5);
    }
</style>
