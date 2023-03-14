<script>
    import axios from "axios";
    import { createEventDispatcher, onMount } from "svelte";

    export let user_data;

    const host = "http://127.0.0.1:8000";
    const dispatch = createEventDispatcher();

    var userSearchQ;
    var users = [];
    var originalUsers = [];

    const execUserSearch = (q) => {
        let result = [];
        q = q.toLowerCase();
        originalUsers.forEach((user) => {
            if (
                user.first_name.toLowerCase().includes(q) ||
                user.last_name.toLowerCase().includes(q) ||
                user.user_name.includes(q)
            ) {
                result.push(user);
            }
        });
        users = result;
    };

    $: if (userSearchQ) {
        execUserSearch(userSearchQ);
    } else {
        users = originalUsers;
    }

    async function getUsers() {
        await axios
            .get(`${host}/monchat/users/${user_data.user_id}/`, {
                headers: {
                    "Content-Type": "application/json",
                },
            })
            .then((response) => {
                users = response.data.data;
                originalUsers = users;
            });
    }

    onMount((e) => {
        getUsers();
    });
</script>

<div id="user_list" class="users">
    <div class="side-topbar">
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <span
            class="back-arrow"
            on:click={(e) => dispatch("chats", { sidebar: "chats" })}
        >
            <svg
                viewBox="0 0 24 24"
                height="24"
                width="24"
                preserveAspectRatio="xMidYMid meet"
                class=""
                version="1.1"
                x="0px"
                y="0px"
                enable-background="new 0 0 24 24"
                xml:space="preserve"
                ><path
                    fill="currentColor"
                    d="M12,4l1.4,1.4L7.8,11H20v2H7.8l5.6,5.6L12,20l-8-8L12,4z"
                /></svg
            >
        </span>
    </div>
    <input
        id="user_search"
        class="user-search"
        type="text"
        placeholder="Search for a user"
        bind:value={userSearchQ}
    />
    <ul>
        {#each users as user, index}
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <li
                on:click={(e) => {
                    dispatch("newchat", user);
                }}
                title={user.user_bio}
            >
                <img
                    src={`${host}/media/${user.user_icon}/`}
                    alt={user.user_id}
                />
                <span class="name"
                    ><b>{`${user.first_name} ${user.last_name}`}</b> • {`${user.user_name}`}</span
                >
                <span class="bio-text"
                    >{user.user_bio.length < 50
                        ? user.user_bio
                        : `${user.user_bio.slice(0, 50)}...`}</span
                >
            </li>
        {/each}
    </ul>
</div>

<style>
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
        overflow: hidden;
    }

    .users {
        width: 100%;
        height: 100%;
        padding: 10px 0;
        color: azure;
    }

    .users ul {
        margin: 0;
        padding: 0;
    }

    .users li {
        margin: 0;
        padding: 5px;
        cursor: pointer;
        display: flex;
        flex-wrap: wrap;
        list-style-type: none;
    }

    .users li img {
        border-radius: 50%;
        width: 50px;
        height: 50px;
        margin: 5px;
    }

    .users li .name {
        text-align: center;
        padding: 2px;
        margin-top: 12px;
        left: 75px;
        position: absolute;
    }

    .users li .bio-text {
        left: 75px;
        font-size: 12px;
        margin-top: 35px;
        text-align: center;
        position: absolute;
    }

    .users li:hover {
        background-color: #202c33;
    }

    .user-search {
        margin-left: 26px;
        background-color: #202c338c;
        border: none;
        outline: none;
        color: azure;
        border-radius: 14px;
        width: 85%;
    }

    .back-arrow svg {
        color: azure;
        cursor: pointer;
        margin: 12px;
    }
</style>
