<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>AI Day Trading</title>

<style>
  @font-face {
    font-family: 'DS-Digital';
    src: url('https://fonts.cdnfonts.com/s/11394/DS-DIGI.woff') format('woff');
  }
  body {
    margin: 0;
    background-color: #111;
    color: limegreen;
    font-family: 'DS-Digital', monospace;
  }
  .title {
    position: absolute;
    top: 0; left: 0;
    padding: 10px;
    font-size: 48px;
    white-space: nowrap;
  }
  .search-container {
    position: fixed;
    top: 10px; right: 10px;
    width: 300px;
    z-index: 1000;
    font-family: Arial, sans-serif;
  }
  .search-input {
    width: 100%;
    padding: 8px;
    font-size: 16px;
    border-radius: 5px;
    border: none;
    outline: none;
  }
  .autocomplete-items {
    position: absolute;
    top: 100%; left: 0; right: 0;
    background-color: #222;
    border: 1px solid #333;
    border-top: none;
    max-height: 200px;
    overflow-y: auto;
    border-radius: 0 0 5px 5px;
  }
  .autocomplete-items div {
    padding: 8px;
    cursor: pointer;
  }
  .autocomplete-items div:hover {
    background-color: #444;
  }
</style>
</head>
<body>

<div class="title">Ai Day Trading</div>

<div class="search-container">
  <input id="searchInput" class="search-input" type="text" placeholder="Search stock...">
  <div id="autocomplete-list" class="autocomplete-items"></div>
</div>

<script>
  // Placeholder: this should be populated from your server/API call
  const instruments = [
    { ticker: 'AAPL_US_EQ', name: 'Apple Inc.' },
    { ticker: 'GOOGL_US_EQ', name: 'Alphabet Inc.' },
    { ticker: 'MSFT_US_EQ', name: 'Microsoft Corp.' },
    // ...more instruments from Trading212
  ];

  const input = document.getElementById('searchInput');
  const list = document.getElementById('autocomplete-list');

  input.addEventListener('input', () => {
    list.innerHTML = '';
    const val = input.value.trim().toLowerCase();
    if (!val) return;

    const matches = instruments.filter(inst =>
      inst.ticker.toLowerCase().includes(val) ||
      inst.name.toLowerCase().includes(val)
    ).slice(0, 10);

    matches.forEach(inst => {
      const item = document.createElement('div');
      item.textContent = `${inst.ticker} — ${inst.name}`;
      item.addEventListener('click', () => {
        input.value = inst.ticker;
        list.innerHTML = '';
      });
      list.appendChild(item);
    });
  });

  document.addEventListener('click', e => {
    if (e.target !== input) list.innerHTML = '';
  });
</script>

</body>
</html>

