<script>
  import AgoraRTM from "agora-rtm-sdk";
  import { onMount } from "svelte";
  import { v4 as uuidv4 } from "uuid";

  const APP_ID = "452f99a0814b44d29d9a446ec20356fc";
  const CHANNEL = "wdj";

  let client = AgoraRTM.createInstance(APP_ID);
  let uid = uuidv4();
  let text = "";
  let messagesRef;
  let messages = [];

  // set onMount
  let channel;

  const appendMessage = message => {
    messages = [...messages, message];
    requestAnimationFrame(() => {
      messagesRef.scrollTop = messagesRef.scrollHeight;
    });
  };

  onMount(async () => {
    await client.login({ uid, token: null });
    channel = await client.createChannel(CHANNEL);
    await channel.join();
    channel.on("ChannelMessage", (message, peerId) => {
      appendMessage({
        text: message.text,
        uid: peerId
      });
    });
  });

  function sendMessage() {
    if (text === "") return;
    channel.sendMessage({ text, type: "text" });
    appendMessage({
      text: text,
      uid
    });
    text = "";
  }
</script>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;

    background: rgb(130, 209, 255);
    background: linear-gradient(
      90deg,
      rgb(174, 220, 238) 0%,
      rgba(194, 180, 217, 1) 49%,
      rgba(148, 187, 233, 1) 100%
    );
  }

  main {
    text-align: center;
    padding: 1em;
    margin: 0 auto;
  }

  .panel {
    display: flex;
    flex-direction: column;
    padding: 20px;
    margin: 0 auto;
    max-width: 300px;
    height: 300px;
    background: rgba(255, 255, 255, 0.25);
    box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
    backdrop-filter: blur(4px);
    border-radius: 10px;
    border: 1px solid rgba(255, 255, 255, 0.18);
  }

  .messages {
    height: 100%;
    width: 100%;
    overflow-y: scroll;
    border-top-left-radius: 5px;
    border-top-right-radius: 5px;
    background-color: white;
  }

  .inner {
    padding: 10px;
  }

  .message {
    text-align: left;
    display: flex;
    margin-bottom: 6px;
  }

  .user-self {
    color: green;
  }

  .user-them {
    color: red;
  }

  form {
    position: relative;
    display: flex;
  }

  input {
    width: 100%;
    border: none;
    height: 20px;
    padding: 8px;
    border-top: 1px solid #999;
    border-radius: 0px;
    outline: none;
  }

  button {
    border: none;
    outline: none;
    background: none;
    position: absolute;
    right: 3px;
    top: 4px;
    font-size: 24px;
  }
</style>

<main>
  <div class="panel">
    <div class="messages" bind:this={messagesRef}>
      <div class="inner">
        {#each messages as message}
          <div class="message">
            {#if message.uid === uid}
              <div class="user-self">You:&nbsp;</div>
            {:else}
              <div class="user-them">Them:&nbsp;</div>
            {/if}
            <div class="text">{message.text}</div>
          </div>
        {/each}
      </div>
    </div>

    <form on:submit|preventDefault={sendMessage}>
      <input bind:value={text} />
      <button>+</button>
    </form>
  </div>
</main>
