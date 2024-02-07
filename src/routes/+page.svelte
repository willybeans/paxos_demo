<script>
  import { onMount } from "svelte";

	let sockets = [];
  let status = ['dead', 'dead', 'dead', 'dead', 'dead'];

	onMount(() => {
    for (let i = 4001; i<4006; i++) {
      const socket = new WebSocket(`ws://localhost:${i}`)
		  socket.addEventListener('open', () => {
	  		console.log('Opened');
  		});

      socket.onmessage = (event) => {
        console.log(event.data)
        if (event.data.slice(6) === 'leader') {
          if (event.data.slice(-4) == i) {
            status[i-4001] = 'leader'
          } else {
            status[i-4001] = 'calm'
          }
        } else {
          status[i-4001] = 'panik'
        }
      }

      sockets.push(socket)
      console.log(sockets)
    }

	});

  function statusUpdate() {
    for (let i = 0; i < 5; i++) {
      if (sockets[i]) {
        console.log(i)
        sockets[i].send('status')
      } else {
        status[i] = 'dead'
      }
    }
  }

</script>

<main>
<h2>Welcome to <s>the Thunderdome</s> Paxos</h2>

{#if (status[0] === 'null')}
a
{:else if (status[0] === 'leader')}
b
{:else if (status[0] === 'dead')}
d
{:else}
c
{/if}


<div class="row">
  {#each Array.from(Array(5).keys()) as column (column)}
    {@const aStatus = status[column]}
    {@const panik = aStatus === 'null'}
    {@const leader = aStatus === 'leader'}
    {@const dead = aStatus === 'dead'}
    <div class:leader class:panik class:dead>
      <button on:click={statusUpdate} class:leader class:panik class:dead></button>
    </div>
  {/each}
  </div>

</main>

<style>
  button {
    background-color: green;
    height: 5rem;
    width: 9rem;
    margin: 0.5rem;
  }

  .leader button {
    background-color: purple
  }

  .panik button{
    background-color: red
  }

  .dead button{
    background-color: black
  }
</style>
