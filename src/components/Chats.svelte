<script>
  import axios from "axios";
  import { createEventDispatcher } from "svelte";
  import ListItem from "./ListItem.svelte";

  export let user_data;
  export let chatList;
  export let originalChatList;

  const host = "http://127.0.0.1:8000";
  const dispatch = createEventDispatcher();

  function handleChatClick(chat) {
    dispatch("chatclick", {
      ...chat,
    });
  }

  var filter = false;
  const filterChats = (e) => {
    document.getElementById("filter").style.backgroundColor = filter
      ? "transparent"
      : "#00a884";
    document.getElementsByClassName("filter")[0].style.color = filter
      ? "#f0ffff80"
      : "azure";
    if (filter) {
      chatList = originalChatList;
      filter = false;
    } else {
      let unreadFilter = [];
      originalChatList.forEach((chat) => {
        if (chat.unread_count > 0) {
          unreadFilter.push(chat);
        }
      });
      chatList = unreadFilter;
      filter = true;
    }
  };

  const execChatSearch = (q) => {
    let result = [];
    q = q.toLowerCase();
    if (!q) {
      return;
    }
    originalChatList.forEach((chat) => {
      if (
        chat.msg_body.toLowerCase().includes(q) ||
        (chat.msg_recipient.first_name.toLowerCase().includes(q) &&
          chat.direction == "outbound") ||
        (chat.msg_recipient.last_name.toLowerCase().includes(q) &&
          chat.direction == "outbound") ||
        (chat.msg_recipient.user_name.toLowerCase().includes(q) &&
          chat.direction == "outbound") ||
        (chat.msg_sender.first_name.toLowerCase().includes(q) &&
          chat.direction == "inbound") ||
        (chat.msg_sender.last_name.toLowerCase().includes(q) &&
          chat.direction == "inbound") ||
        (chat.msg_sender.user_name.toLowerCase().includes(q) &&
          chat.direction == "inbound")
      ) {
        result.push(chat);
      }
    });
    if (q) {
      console.log("res", result);
      chatList = result;
    } else {
      console.log("none", originalChatList);
      chatList = originalChatList;
    }
  };

  function showSettings(e) {
    // settings page
  }

  function logout(e) {
    window.localStorage.setItem("monchat_user_id", "");
    window.location.reload();
    dispatch("logout", {});
  }

  let chatSearchQ = "";
  $: {
    execChatSearch(chatSearchQ);
  }

  function hide_menu(e) {
    let drop_down = document.getElementById("menu_dropdown");
    let drop_down_list = document.getElementById("menu_list");

    let not_found = true;
    if (drop_down_list) {
      drop_down_list.childNodes.forEach((node) => {
        if (node === e.target) {
          not_found = false;
        }
      });
    }

    if (not_found && drop_down) {
      drop_down.style.opacity = "0";
      setTimeout((e) => {
        drop_down.style.display = "none";
      }, 1200);
      setTimeout((e) => {
        document.removeEventListener("click", hide_menu);
      }, 500);
    } else {
      document.removeEventListener("click", hide_menu);
    }
  }

  function show_menu(e) {
    let drop_down = document.getElementById("menu_dropdown");
    drop_down.style.opacity = "1";
    drop_down.style.display = "block";

    setTimeout((e) => {
      document.addEventListener("click", hide_menu);
    }, 500);
  }
</script>

<div id="chats">
  <div class="side-topbar">
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <img
      class="profile-img"
      id="profile_btn"
      src={`${host}/media/${user_data.user_icon}/`}
      alt={"profile"}
      on:click={(e) => {
        dispatch("profile", {
          sidebar: "profile",
        });
      }}
    />
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <span class="dot-menu" on:click={show_menu}>
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

  <div class="menu-dropdown" id="menu_dropdown">
    <ul id="menu_list">
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <li
        on:click={(e) => {
          dispatch("newgroup", {
            sidebar: "new_group_choice",
          });
        }}
      >
        New Group
      </li>
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <li on:click={showSettings}>Settings</li>
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <li on:click={logout}>Logout</li>
    </ul>
  </div>
  <div class="search-bar">
    <svg
      width="24px"
      height="24px"
      viewBox="0 0 24 24"
      fill="#f0ffff80"
      xmlns="http://www.w3.org/2000/svg"
    >
      <path
        d="M11.01 20.02C15.9861 20.02 20.02 15.9861 20.02 11.01C20.02 6.03391 15.9861 2 11.01 2C6.03391 2 2 6.03391 2 11.01C2 15.9861 6.03391 20.02 11.01 20.02Z"
      />
      <path
        d="M21.9901 18.95C21.6601 18.34 20.9601 18 20.0201 18C19.3101 18 18.7001 18.29 18.3401 18.79C17.9801 19.29 17.9001 19.96 18.1201 20.63C18.5501 21.93 19.3001 22.22 19.7101 22.27C19.7701 22.28 19.8301 22.28 19.9001 22.28C20.3401 22.28 21.0201 22.09 21.6801 21.1C22.2101 20.33 22.3101 19.56 21.9901 18.95Z"
      />
    </svg>
    <input
      type="text"
      class="chat-search"
      bind:value={chatSearchQ}
      placeholder="Search or start a new chat"
    />
  </div>
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <span id="filter" on:click={filterChats}>
    <svg
      class="filter"
      viewBox="0 0 24 24"
      height="22"
      width="22"
      preserveAspectRatio="xMidYMid meet"
      version="1.1"
      x="0px"
      y="0px"
      enable-background="new 0 0 24 24"
      xml:space="preserve"
    >
      <path
        fill="currentColor"
        d="M10,18.1h4v-2h-4V18.1z M3,6.1v2h18v-2H3z M6,13.1h12v-2H6V13.1z"
      />
    </svg>
  </span>

  {#if chatList.length > 0}
    {#each chatList as chat}
      <ListItem {...chat} clickCallback={(e) => handleChatClick(chat)} />
    {/each}
  {:else}
    <span class="empty"><i>No recent chats, kindly start a chat.</i></span>
  {/if}

  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <span
    class="new-contact"
    on:click={(e) => {
      dispatch("users", {
        sidebar: "users",
      });
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
        enable-background="new    "
        d="M19.005,3.175H4.674C3.642,3.175,3,3.789,3,4.821V21.02 l3.544-3.514h12.461c1.033,0,2.064-1.06,2.064-2.093V4.821C21.068,3.789,20.037,3.175,19.005,3.175z M14.016,13.044H7.041V11.1 h6.975V13.044z M17.016,9.044H7.041V7.1h9.975V9.044z"
      /></svg
    >
  </span>
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

  .dot-menu {
    right: 8px;
    top: 12px;
    color: azure;
    cursor: pointer;
    position: absolute;
  }

  .empty {
    color: rgba(240, 255, 255, 0.5);
    margin-top: 20%;
    left: 40px;
    position: absolute;
    width: 300px;
    z-index: 4;
  }

  .profile-img {
    width: 40px;
    height: 40px;
    margin: 5px;
    border-radius: 50%;
    margin-left: 10px;
  }

  .search-bar {
    width: 85%;
    padding: 0;
    margin-left: 10px;
    border-radius: 15px;
    display: inline-block;
    margin-bottom: 10px;
    background-color: #202c33;
  }
  .search-bar svg {
    color: #f0ffff80;
    font-size: 7px;
    margin: 3px;
    position: absolute;
  }
  .chat-search {
    width: 86%;
    margin: 0;
    margin-left: 27px;
    background-color: #202c33;
    border: none;
    color: #f0ffff;
    outline: none;
  }

  .filter {
    color: #f0ffff80;
    cursor: pointer;
  }

  #filter {
    position: absolute;
    border-radius: 50%;
    padding: 2px;
    width: 25px;
    height: 25px;
    margin-left: 10px;
    text-align: center;
  }

  .menu-dropdown {
    width: 210px;
    padding: 10px 0;
    background-color: #202c33;
    border-radius: 6px;
    margin: 0;
    position: absolute;
    box-shadow: -2px 5px 4px 1px rgba(0, 0, 0, 0.482);
    left: 45%;
    top: 5px;
    color: azure;
    z-index: 10;
    display: none;
    transition: opacity 1s ease;
  }

  .new-contact {
    background-color: #00a884;
    color: azure;
    font-size: 40px;
    font-weight: bold;
    text-align: center;
    width: 50px;
    height: 50px;
    position: fixed;
    bottom: 20px;
    left: 24%;
    cursor: pointer;
    box-shadow: 5px 3px 4px 2px rgba(0, 0, 0, 0.482);
    border-radius: 50%;
  }

  .menu-dropdown ul {
    margin: 0;
    padding: 0;
  }

  .menu-dropdown li {
    padding: 7px;
    cursor: pointer;
    list-style-type: none;
  }
  .menu-dropdown li:hover {
    background-color: #111b21;
  }
</style>
