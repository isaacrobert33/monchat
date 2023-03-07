<script>
    import axios from 'axios';
    import { createEventDispatcher } from 'svelte';

    let sign_in = true;
    let sign_up = false;
    const host = "http://127.0.0.1:8000/monchat";

    const dispatch = createEventDispatcher();

    $: if (sign_up === true) {
        sign_in = false;
    }

    function show_signup() {
        sign_up = true;
    }

    let signin_err = false;

    async function signIn() {
        let payload = {
            uname: document.getElementById('login_uname').value, pwd: document.getElementById('login_pwd').value
        };
        await axios.post(`${host}/login/`, payload, {
                headers: {
                    'Content-Type': 'application/json'
                }
            }
        ).then(
            response => {
                console.log(response);
                dispatch('login', {
                    ...response.data.data
                })
            }
        ).catch(
            err => {
                console.log("Error in sigin")
                signin_err = true;
            }
        )
    }

    
    async function signUp() {
        let payload = {
            uname: document.getElementById('sign_up_uname').value, pwd: document.getElementById('sign_up_pwd').value
        };
        await axios.post(`${host}/sign_up/`, payload, {
                headers: {
                    'Content-Type': 'application/json'
                }
            }
        ).then(
            response => {
                console.log(response);
                dispatch('login', {
                    ...response.data.data
                })
            }
        )
    }

    let invalid_data = false;
    let pwd_entries;

    $: if (document.getElementById("sign_up_pwd") && document.getElementById("sign_up_pwd").value !== pwd_entries) {
            invalid_data = true;
        } else {
            invalid_data = false;
        }
    

    let uname_invalid = false;
    const check_username = async () => {
        await axios.get(`${host}/check_username/${document.getElementById("sign_up_uname").value}/`, {
            headers: {
                    'Content-Type': 'application/json'
                }
        }).then(
            response => {
                console.log(response);
                if (response.data.data.exists) {
                    uname_invalid = true;
                    invalid_data = true;
                } else {
                    uname_invalid = false;
                    invalid_data = false;
                }
            }
        )
    }

    var timeID;
    let inputU;
    $: {
        if (inputU) {
            if (timeID){clearTimeout(timeID)}
            timeID = setTimeout(check_username, 500)
        }
        
    }

</script>

<div class="login blur-overlay">
    {#if sign_in}
        <div class="user-box">
            <h1><i>MonChat</i></h1>
            {#if signin_err}
                <span style="color: tomato; font-size: 13px;"><i>The user name or password was incorrect</i></span>
            {/if}
            <input id="login_uname" type="text" class="user_name" placeholder="Enter your user name"/>
            <input id="login_pwd" type="password" class="pwd" placeholder="Enter your password"/>
            <button on:click={signIn}>Sign in</button>
            <span class="signup_link">or <a href="#sign_up" on:click={show_signup}>sign up</a></span>
        </div>
    {:else if sign_up}
        <div class="user-box">
            <h1><i>MonChat</i></h1>
            <input id="sign_up_uname" type="text" class="user_name" placeholder="Enter a user name" bind:value={inputU}/>
            <input id="sign_up_pwd" type="password" class="pwd" placeholder="Create your password" />
            <input id="sign_up_pwd_2" type="password" class="pwd" placeholder="Re-Enter your password" bind:value={pwd_entries}/>
            {#if invalid_data}
                <span style="color: tomato; font-size: 13px;"><i>Password mismatch</i></span>
            {:else if uname_invalid}
                <span style="color: tomato; font-size: 13px;"><i>Username taken, try another.</i></span>
            {/if}
            <button id="sign_up-btn" on:click={signUp} disabled={invalid_data}>Sign up</button>
        </div>
    {/if}

</div>

<style>
    .login {
        z-index: 5;
        display: grid;
        align-items: center;
        overflow: hidden;
        color: azure;
    }
    .blur-overlay {
        backdrop-filter: blur(5px);
        position: fixed;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        background-color: rgba(240, 255, 255, 0.2);
    }
    .user-box {
        width: 20%;
        height: 20%;
        background-color: #202c33;
        border-radius: 12px;
        z-index: 6;
        display: grid;
        align-items: center;
        padding: 20px 20px;
        height: 300px;
        margin-left: 34%;
        box-shadow: 0px 1px 6px 2px rgba(0, 0, 0, 0.4);
    }

    h1 {
        text-align: center;
    }

    .signup_link {
        margin-left: 200px;
        font-size: 16px;
    }

    input {
        outline: none;
        color: azure;
        border: none;
        border-radius: 8px;
        margin-bottom: 10px;
        background-color: #2a3942;
    }

    button {
        background-color: #005c4b;
        border: none;
        padding: 5px;
        color: azure;
        border-radius: 5px;
    }

    a {
        color: #005c4b;
    }
</style>