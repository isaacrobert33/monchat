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

    $: if (sidebar === "profile") {
        if (mutated) {
            axios
                .get(`${host}/monchat/user/${user_data.user_id}/`, {
                    headers: { "Content-Type": "application/json" },
                })
                .then((response) => {
                    user_data = response.data.data;
                });
        }

        setTimeout(() => {
            let fileBtn = document.getElementById("file_input");
            let profileBtn = document.getElementById("profile-bg");
            profileBtn.addEventListener("click", () => {
                fileBtn.click();
            });
            document.getElementById("fname").focus();
        }, 2000);
    } else if (sidebar === "chats" && mutated) {
        let payload = {
            fname: document.getElementById("fname").value,
            lname: document.getElementById("lname").value,
            user_bio: document.getElementById("bio").value,
        };
        axios
            .put(`${host}/monchat/user/${user_data.user_id}/`, payload, {
                headers: { "Content-Type": "application/json" },
            })
            .then((response) => {
                // alert(response.data.msg);
            })
            .catch((err) => {
                alert(err);
            });
    } else if (sidebar == "new_group") {
        setTimeout((f) => {
            let fileIn = document.getElementById("group_file_input");
            let btn = document.getElementById("group_icon_btn");
            btn.addEventListener("click", (e) => {
                fileIn.click();
            });
        }, 2000);
    } else {
        // getUsers();
    }
</script>

<div class="list-container">
    {#if sidebar === "profile"}
        <Profile
            {user_data}
            {mutated}
            on:onback={(e) => {
                sidebar = "chats";
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
            }}
            on:back={(e) => {
                sidebar = "chats";
            }}
        />
    {:else if sidebar == "new_group"}
        <NewGroup
            on:back={(e) => {
                sidebar = "chats";
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
