
<script>
  import { cubicOut } from 'svelte/easing';
	import { fade, crossfade } from 'svelte/transition';
  import { flip } from 'svelte/animate';
  import Hamburger from '$lib/Hamburger.svelte';
  import Menu from '$lib/Menu.svelte';
  import Help from '$lib/Help.svelte';

	const [send, receive] = crossfade({
		duration: d => Math.sqrt(d * 200),
		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;

			return {
				duration: 250,
				easing: cubicOut,
				css: t => `
					transform: ${transform} scale(${t});
          opacity: ${t};
				`
			};
		}
	});

  import { onMount } from 'svelte';
  import getEditDistance from '$lib/levenshtein';
  import download from '$lib/download';

  let pasteSide = 'left';
  let searchText = '';
  let searching = false;
  let menuOpen = false;
  let showHelp = false;

  let leftCards = [
    {id: 0, name: 'B.B. King'},
    {id: 1, name: 'Robert Johnson'},
    {id: 2, name: 'Jimi Hendrix'},
  ];
  let rightCards = [
    {id: 3 , name: 'Riley King'}, 
    {id: 4, name: 'Jimi Hendrix'},
  ];

  $: filteredRightCards = rightCards.filter(card => {
    if (!searching) return true;
    if (!card.name) return console.log('card with missing name', rightCards);
    return card.name.toLowerCase().match(searchText.toLowerCase());
  });

  $: numMatches = Object.values(matching).filter(Boolean).length;

  let uid = 5;
  let currentCard = 0;
  let matching = {};

  function setCard(i) {
    const leftId = leftCards[i].id;
    if (i != currentCard) {
      currentCard = i;
      rightCards = rightCards.sort((r1, r2) => 
        getEditDistance(leftCards[i].name, r1.name) - 
        getEditDistance(leftCards[i].name, r2.name)
      );
      focus();
    } else if (matching[leftId]) {
      rightCards = [matching[leftId], ...rightCards];
      matching[leftId] = undefined;
    }
  }

  function makeMatch(j) {
    const leftId = leftCards[currentCard].id;
    if (matching[leftId]) {
      // already a match; we have to pull it out
      const oldCard = matching[leftId]; 
      matching[leftId] = rightCards[j];
      const remainingCards = rightCards.slice(0, j).concat(rightCards.slice(j + 1));
      rightCards = [oldCard, ...remainingCards];
    } else {
      matching[leftId] = rightCards[j];
      rightCards = rightCards.slice(0, j).concat(rightCards.slice(j + 1));
    }
  }

  function reset() {
    currentCard = 0;
    rightCards = Object.values(matching).filter(Boolean).sort((a, b) => a.id - b.id).concat(rightCards);
    matching = {};
  }

  function handleCopy(event) {
    const rightCol = leftCards.map(card => {
      if (matching[card.id] && matching[card.id].name) {
        return matching[card.id].name;
      } else {
        return "";
      }
    }).join('\n');
    event.clipboardData.setData('text/plain', rightCol);
    event.preventDefault()
  }

  function makeNames(clipboardContents) {
    return clipboardContents
        .split('\n')
        .map(name => ({
          id: uid++,
          name,
        }));
  }

  function handlePaste(event) {
    const contents = event.clipboardData.getData('text/plain');
    if (pasteSide === 'left') {
      leftCards = makeNames(contents)
      pasteSide = 'right';
      reset();
    } else {
      rightCards = makeNames(contents);
      pasteSide = 'left';
      reset();
    }
  }

  function exportData() {
    const data = leftCards.map((record, i) => 
      record.name + "," + (
        (matching[record.id] && matching[record.id].name) || ""
      )
    ).join("\n");
    download({
      filename: "matching.csv", 
      data, 
      type: "csv"
    });
  }

  function focus() {
    setTimeout( () => {
      const searchBox = document.getElementById("search")
      if (searchBox) searchBox.focus();
    }, 5);
  }

  const digits = new Set(["1", "2", "3", "4", "5", "6", "7", "8", "9"]);

  function handleKeydown(event) {
    if (event.key === 'ArrowUp' && 
          currentCard >= 1) {
      setCard(currentCard - 1);
      focus();
    } else if (event.key === 'ArrowDown' && 
               currentCard < leftCards.length - 1) {
      setCard(currentCard + 1);
      focus();
    } else if (searching) {
      if (event.key === "Escape") {
        showHelp = false;
        searching = false;
        searchText = '';
      } else if (event.key === "Enter" && filteredRightCards.length > 0) {
        makeMatch(rightCards.indexOf(filteredRightCards[0]));
        searchText = '';
        if (currentCard < leftCards.length - 1) {
          setCard(currentCard + 1);
        }
      }
    } else if (digits.has(event.key)) {
      const selection = parseInt(event.key) - 1;
      if (selection < rightCards.length) {
        makeMatch(selection);
      }
      if (currentCard < leftCards.length - 1) {
        setCard(currentCard + 1);
      }
    } else if (event.key === 'e') {
      exportData();
    } else if (event.key === "Escape") {
      reset();
    } else if (event.key === "/") {
      searching = true;
      focus();
    }
  }

  function toggleHelp () {
    showHelp = !showHelp;
  }

  function toggleSearch() {
    searching = !searching;
  }

  onMount(() => {
    window.addEventListener('paste', handlePaste)
    window.addEventListener('copy', handleCopy)
    window.addEventListener('keydown', handleKeydown)
  });
</script>

<style>
  h1 { 
    margin-top: 100px;
    margin-bottom: 40px;
  }

  .grid-container {
    display: grid;
    justify-content: center;
    column-gap: 22px;
    grid-template-columns: 400px;
  }

  .column {
    border-radius: 4px;
  }

  .name {
    cursor: pointer;
    border: 1px solid white;
    padding: 16px;
    border-radius: 2px;
    margin-bottom: 10px;
    display: grid;
    grid-template-columns: 100%;
    text-align: center;
    align-content: center;
  }

  .name.active {
    position: relative;
    border: 1px solid #f9ff53;
    background-color: #f9ff5311;
    transform: scale(1.02);
    transition: transform 0.25s;
  }

  .right-names {
    position: absolute;
    top: 0;
    right: 0;
    transform: translate(calc(100% + 20px), 0);
  }

  .right-name {
    margin-bottom: 5px;
    cursor: pointer;
    transition: transform 0.3s;
  }

  .right-name:hover {
    transform: scale(1.05);
  }

  .centered {
    text-align: center;
  }

  .digit {
    margin-right: 4px;
    color: #AAA;
  }

  .searching {
    position: absolute;
    top: -4rem;
    right: 0;
    padding: 1rem;
    transform: translate(100%, 0);
  }

  .searching input {
    background: transparent;
    padding: 10px;
    border: 1px solid white;
    color:white;
    border-radius: 4px;
  }

  @media only screen and (max-width: 825px) {
    .right-names  {
      position: relative;
      transform: none;
      margin-top: 25px;
    }

    .searching {
      position: relative;
      top: 0;
      transform: none;
      margin-bottom: -30px;
    }
  }

  input:focus {
    outline: none;
  }

  .match-count {
    margin: 40px;
  }

  nav {
    display: flex;
    width: 100%;
    height: 60px;
    /* border-bottom: 1px solid #EEE; */
    align-items: center;
  }

  .blur {
    filter: blur(8px);
  }
</style>

{#if showHelp}
  <Help {toggleHelp}/>
{/if}

<div class={showHelp ? "blur" : ""}>
  <nav>
    <Hamburger bind:open={menuOpen}/>
    <Menu open={menuOpen} {exportData} {toggleHelp} {toggleSearch} />
  </nav>
  <h1 class="centered">Fuzzy Join</h1>
  <div class="match-count centered">
    {numMatches} match{numMatches !== 1 ? "es" : ""}
  </div>
  <div class="grid-container">
    <div class="column">
      {#each leftCards as card, i}
        <div class={"left name" + (i === currentCard ? " active" : "")}
          on:click={() => setCard(i)}>
          <span>
            { card.name } 
            {#if matching[leftCards[i].id]}
              ⟷
              <span 
                class="on-card-match"
                in:receive={{key: matching[leftCards[i].id].id}}
                out:fade={{duration: 75}}
                >
                { matching[leftCards[i].id].name }
              </span>
            {/if}
            {#if i === currentCard}
              {#if searching}
                <div class="searching">
                  <input bind:value={searchText} id="search"/>
                </div>
              {/if}
              <div class="right-names">
                {#each filteredRightCards as rightCard, j (rightCard.id)}
                  <div class="right-name"
                    animate:flip
                    out:send={{key: rightCard.id}}
                    in:fade|local
                    on:click|stopPropagation={ () => 
                      makeMatch(j)
                    }> 
                    <span class="digit">{ j + 1 }</span>
                    { rightCard.name } 
                  </div>
                {/each}
              </div>
            {/if}
          </span>
        </div>
      {/each}
    </div>
  </div>
</div>