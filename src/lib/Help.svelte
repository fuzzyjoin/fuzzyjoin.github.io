<script>
  export let toggleHelp;
</script>

<style>
  .help {
    position: absolute;
    overflow-y: scroll;
    top: 60px;
    left: 60px;
    bottom: 60px;
    right: 60px;
    background-color: white;
    color: #322a4e;
    z-index: 20;
    padding: 30px;
  }

  .mask {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    opacity: 0.2;
    filter: blur(8px);
    background: white;
    z-index: 10;
  }
</style>

<div class="mask" on:click={toggleHelp}/>
<div class="help">
  <h3>Introduction</h3>
  <p>Welcome to Fuzzy Join! This is a simple tool to help you match values in two spreadsheet columns.</p>
    
  <p>For example, suppose you have two lists of names and want to figure out which folks are in both lists. If each such person's name is not printed exactly the same in the two lists, you'll have to painstakingly read through the lists and match names up as you go. Fuzzy Join helps with that.</p>

  <h3>Instructions</h3>
  <p>Begin by copying the first column from your spreadsheet software and then do a paste action with this browser window open. Repeat with the second column.</p>

  <p>Next you'll see a list of suggestions for the first entry in the first column. Choose a match by clicking on it or by pressing the corresponding digit on the keyboard. You can also search to filter possible matches by pressing <tt>/</tt>. Continue until all matches have been found. Export as a CSV or perform a copy action to put the matching on your clipboard.</p>

  <h3>Security</h3>
  <p>Fuzzy Join is secure because your data stays entirely within your browser.

</div>