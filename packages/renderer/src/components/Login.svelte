<script>
import _ from "lodash"; 
import { get } from 'svelte/store';
import { data } from "../data/localStore.js";
import { onMount } from 'svelte';

let selected;

let globalData = get(data);

async function login(username) {
    let data = JSON.parse(await window.please.get("login"));
    let result = data.type;
    if (result == "Cancelled") {
        console.log("Sign-in Cancelled!")
    }
    else if (result == "Success") {
        username = data.profile.name;

        const userSnippet = { username, data }
        globalData.users =  await window.please.get(
            "refreshUsers",
            _.reject(_.compact(_.concat(globalData.users, userSnippet)), _.isEmpty)
            );
        console.log(globalData)
        globalData.selected = username;
        globalData.selectedIndex = globalData.users.length - 1;
        console.log("Sign-in Successful!");
    } 
}

const select = async (index) => {
    globalData.selected = globalData.users[index].username;
    globalData.selectedIndex = index;
    globalData;
}

const logout = async (index) => {
    if (globalData.selected == globalData.users[index].username) {
        if (globalData.users.length > 1) {
            globalData.selected = globalData.users[1].username;
            globalData.selectedIndex = 1;
        } else {
            globalData.selected = "e";
        }
    }
    delete globalData.users[index];
    globalData.users = _.compact(globalData.users);
}

$: data.update((thing) => thing = globalData);

onMount( async () => {
    const validatedAndRefreshedUsers = 
    await window.please.get("refreshUsers", globalData.users);
    globalData.users = validatedAndRefreshedUsers
});

</script>

<main>
    <!-- <button 
    class:noLogin="{globalData.selected === 'e'}"
    class="login" on:click={
        async () => {
            await login()
        }
    }>ADD LOGIN</button><br> -->
    <!-- <img class="inline bodyIMG" src="https://mc-heads.net/body/{selected}" alt="Your Minecraft Body"/> -->
    {#key globalData.users}
        {#if (JSON.stringify(globalData.users) !== "{}") && (globalData.users)}
            {#each globalData.users as user, i}
            <div class="inline" class:selected="{user.username == globalData.selected}">
                <img class="userHead" alt="Minecraft Head" src="https://mc-heads.net/avatar/{user.username}/180.png"/>
                <button class="user" on:click={
                    async () => {
                        select(i)
                    }
                }>{user.username}</button>
                <button class="logout" on:click={
                    async () => {
                        logout(i)
                    }
                }><img class = "logoutImage" alt="Logout" src="images/xsymb.webp"/></button>
                <br>
            </div>
            {/each}
        {/if}
    {/key}
    <div class="inline">
        <button class="user" on:click={
            async () => {
                await login();
            }
        }>Log into an account</button>
        <button class="logout" on:click={
            async () => {
                await login();
            }
        }>+</button>
        <br>
    </div>
    
</main>

<!-- <canvas id="skin_container"></canvas> -->

<style>
    /* .bodyIMG {
        width: 30%;
        margin-left: auto;
        margin-right: auto;
        float: left;
    } */
    .logoutImage {
        filter: invert(100%);
        opacity: 0.5;
        width: 20px;
        margin: auto;
        display: block;
    }
    .inline {
        border-radius: 5px 5px 5px 5px;
        display: flex;
        padding: 7px;
        margin: 5px;
        background-color: #181816;
        transition-timing-function: ease-in-out;
        color: rgba(255, 255, 255, 0.503);
        transition: all .8s;
        
    }
    .selected {
        color: rgba(255, 255, 255, 0.803);
        background-color: rgb(59, 59, 59);
    }
    .logout {
        border-radius: 5px 5px 5px 5px;
        float: right;
        flex: 1 0;
        height: 50px;
        width: 50px;
        aspect-ratio: 1 / 1;
        background-color: black;
        padding: 5px;
        text-align: center;
    }
    .logout:hover {
        color: white;
    }
    .user {
        flex: 10 0;
        padding: 5px;
        text-align: left;
        font-size: 20px;
    }
    .user:hover {
        background-color: #08b2e100;
        color: white;
    }
    .noLogin {
        font-size: 20px;
        padding: 10px;
        background-color: rgba(255, 0, 0, 0.287);
    }
    .userHead {
        border-radius: 5px 5px 5px 5px;
        height: 50px;
        aspect-ratio: 1 / 1;
    }
</style>