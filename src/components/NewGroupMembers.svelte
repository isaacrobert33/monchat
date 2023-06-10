<script>
  import axios from "axios";
  import { createEventDispatcher } from "svelte";
  import { onMount } from "svelte";

  export let user_data;

  var users = [];

  const host = process.env.API_URL;
  const dispatch = createEventDispatcher();
  var users4group = [];
  async function getUsers() {
    await axios
      .get(`${host}/monchat/users/${user_data.user_id}/`, {
        headers: {
          "Content-Type": "application/json",
        },
      })
      .then((response) => {
        users = response.data.data;
        users4group = users;
      });
  }

  onMount((e) => {
    getUsers();
  });

  var newGroupUsers = [];
  var usersIndex = {};
</script>

<div id="new_group" class="new-group">
  <div class="side-topbar">
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <span
      class="back-arrow"
      on:click={(e) => {
        dispatch("back", { sidebar: "chats" });
      }}
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
    <h3>Add the group members</h3>
  </div>
  <div class="group-members">
    {#each newGroupUsers as user}
      <span class="new-member">
        <img src={`${host}/media/${user.user_icon}/`} alt={user.user_id} />
        <span class="member-name">{`${user.first_name} ${user.last_name}`}</span
        >
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <span
          class="member-cancel"
          on:click={(e) => {
            let n = [...newGroupUsers];
            n.splice(newGroupUsers.indexOf(user), 1);
            newGroupUsers = n;
            users4group.splice(usersIndex[user.user_name], 0, user);
            users4group = users4group;
          }}>&times;</span
        >
      </span>
    {/each}
  </div>
  <input class="user-search" placeholder="Enter user's name" />
  <ul>
    {#each users4group as user, index}
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <li
        on:click={(e) => {
          newGroupUsers = [...newGroupUsers, user];
          let n = users4group;
          n.splice(users4group.indexOf(user), 1);
          users4group = n;
          usersIndex[user.user_name] = index;
        }}
        title={user.user_bio}
      >
        <img src={`${host}/media/${user.user_icon}/`} alt={user.user_id} />
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
  {#if newGroupUsers.length > 0}
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <span
      class="group-btn"
      on:click={(e) => {
        dispatch("newgroup", { members: newGroupUsers });
      }}
    >
      <svg
        viewBox="0 0 30 30"
        height="30"
        width="30"
        preserveAspectRatio="xMidYMid meet"
        class=""
        version="1.1"
        x="0px"
        y="0px"
        enable-background="new 0 0 30 30"
        xml:space="preserve"
        ><path
          fill="currentColor"
          d="M15,7l-1.4,1.4l5.6,5.6H7v2h12.2l-5.6,5.6L15,23l8-8L15,7z"
        /></svg
      >
    </span>
  {/if}
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

  .back-arrow svg {
    color: azure;
    cursor: pointer;
    margin: 12px;
  }

  .new-group {
    width: 100%;
    height: 100%;
    padding: 10px 0;
    color: azure;
  }

  .group-members {
    display: flex;
    flex-wrap: wrap;
  }

  .new-group ul {
    margin: 0;
    padding: 0;
  }

  .new-group li {
    margin: 0;
    padding: 5px;
    cursor: pointer;
    display: flex;
    flex-wrap: wrap;
    list-style-type: none;
  }

  .new-group li img {
    border-radius: 50%;
    width: 50px;
    height: 50px;
    margin: 5px;
  }

  .new-group li .name {
    text-align: center;
    padding: 2px;
    margin-top: 12px;
    left: 75px;
    position: absolute;
  }

  .new-group li .bio-text {
    left: 75px;
    font-size: 12px;
    margin-top: 35px;
    text-align: center;
    position: absolute;
  }

  .new-group li:hover {
    background-color: #202c33;
  }
  .new-member {
    display: flex;
    flex-wrap: wrap;
    margin: 5px;
  }
  .new-member img {
    border-radius: 50%;
    width: 20px;
    height: 20px;
    margin: 2px;
  }

  .new-member .member-name {
    font-size: 13px;
    margin-top: 4px;
  }

  .member-cancel {
    cursor: pointer;
    margin-left: 5px;
    font-size: 19px;
  }

  .group-btn {
    color: azure;
    border-radius: 50%;
    width: 46px;
    height: 46px;
    text-align: center;
    position: fixed;
    bottom: 20px;
    left: 10%;
    background-color: #00a884;
  }
  .group-btn svg {
    margin-top: 8px;
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

  .side-topbar h3 {
    top: 0px;
    left: 70px;
    color: #f0ffff;
    position: absolute;
  }
</style>
