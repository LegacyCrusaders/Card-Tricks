<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Pick a Card & Number</title>
  <style>
    body {
      background-color: black;
      color: white;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      text-align: center;
      margin-top: 50px;
    }
    h1 {
      margin-bottom: 10px; /* reduced space below title */
      cursor: pointer; /* pointer on hover */
      user-select: none;
    }
    button {
      font-size: 18px;
      margin: 15px 0 30px 0;
      padding: 10px 20px;
      background-color: white;
      color: black;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      display: block;
      margin-left: auto;
      margin-right: auto;
      width: 160px;
    }
    button:hover {
      background-color: lightgray;
    }
    #card-display {
      font-size: 140px;
      margin: 20px 0 10px 0;
      user-select: none;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: 10px;
      height: 180px;
    }
    #card-rank {
      font-weight: bold;
      font-size: 140px;
      line-height: 1;
      color: white;
      user-select: none;
    }
    #card-suit {
      font-size: 140px;
      line-height: 1;
      user-select: none;
    }
    #number-display {
      font-size: 40px;
      margin: 30px 0 10px 0;
      user-select: none;
    }
    #stack-display {
      margin-top: 30px;
      font-size: 16px;
      color: #ccc;
      max-width: 90vw;
      white-space: pre-wrap;
      word-wrap: break-word;
      user-select: text;
      display: none; /* hidden by default */
      text-align: left;
      margin-left: auto;
      margin-right: auto;
    }
  </style>
</head>
<body>
  <h1 id="title">Pick a Card & Number</h1>

  <div id="card-display" aria-label="Playing card">
    <div id="card-rank">A</div>
    <div id="card-suit">♠</div>
  </div>
  <button onclick="buttonPressed('card')">Random Card</button>

  <div id="number-display">Number: (loading...)</div>
  <button onclick="buttonPressed('number')">Random Number</button>

  <button onclick="buttonPressed('both')">Random Both</button>

  <pre id="stack-display"></pre>

  <script>
    const stack = [
      '4 of Clubs', '7 of Hearts', 'King of Diamonds', '9 of Spades', '2 of Clubs', 'Jack of Hearts', '5 of Diamonds', 'Ace of Spades',
      '10 of Clubs', '3 of Hearts', '6 of Spades', 'Queen of Clubs', '8 of Diamonds', 'King of Clubs', '2 of Diamonds', '4 of Hearts',
      'Ace of Clubs', '7 of Spades', '3 of Diamonds', '9 of Clubs', '5 of Hearts', 'Jack of Spades', '6 of Diamonds', 'Queen of Hearts',
      '8 of Clubs', 'King of Spades', '2 of Spades', '4 of Diamonds', 'Ace of Hearts', '7 of Clubs', '3 of Spades', '9 of Hearts',
      '5 of Spades', 'Jack of Diamonds', '6 of Clubs', 'Queen of Spades', '8 of Hearts', 'King of Hearts', '2 of Hearts', '4 of Spades',
      'Ace of Diamonds', '7 of Diamonds', '3 of Clubs', '9 of Diamonds', '5 of Clubs', 'Jack of Clubs', '6 of Hearts', 'Queen of Diamonds',
      '8 of Spades', '10 of Diamonds', '10 of Hearts', '10 of Spades'
    ];

    let currentCard = null;
    let currentNumber = null;
    let lastPresses = [];

    // For secret clicks
    let clickCount = 0;
    let clickTimer = null;

    function numberToCard(n) {
      return stack[n - 1];
    }

    function cardToNumber(card) {
      return stack.indexOf(card) + 1;
    }

    function parseCard(cardStr) {
      const [rank, , suit] = cardStr.split(' ');
      return { rank, suit };
    }

    function suitSymbol(suit) {
      switch (suit) {
        case 'Spades':   return { symbol: '♠', color: 'white' };
        case 'Clubs':    return { symbol: '♣', color: 'white' };
        case 'Hearts':   return { symbol: '♥', color: 'red' };
        case 'Diamonds': return { symbol: '♦', color: 'red' };
        default:         return { symbol: '?', color: 'white' };
      }
    }

    function updatePressHistory(buttonName) {
      lastPresses.push(buttonName);
      if (lastPresses.length > 3) lastPresses.shift();
    }

    function displayCard(cardStr) {
      const cardRankDiv = document.getElementById('card-rank');
      const cardSuitDiv = document.getElementById('card-suit');

      const { rank, suit } = parseCard(cardStr);
      const { symbol, color } = suitSymbol(suit);

      let displayRank = rank;
      if (rank === '10') displayRank = '10';
      else if (rank === 'Jack') displayRank = 'J';
      else if (rank === 'Queen') displayRank = 'Q';
      else if (rank === 'King') displayRank = 'K';
      else if (rank === 'Ace') displayRank = 'A';

      cardRankDiv.textContent = displayRank;
      cardSuitDiv.textContent = symbol;
      cardSuitDiv.style.color = color;
      cardRankDiv.style.color = 'white';
    }

    function showCard() {
      if (lastPresses.length > 0 && lastPresses[lastPresses.length - 1] === 'number' && currentNumber !== null) {
        currentCard = numberToCard(currentNumber);
      } else {
        const randomIndex = Math.floor(Math.random() * 52);
        currentCard = stack[randomIndex];
      }
      displayCard(currentCard);
      updatePressHistory('card');
    }

    function showNumber() {
      const numberDisplay = document.getElementById('number-display');
      if (lastPresses.length > 0 && lastPresses[lastPresses.length - 1] === 'card' && currentCard !== null) {
        currentNumber = cardToNumber(currentCard);
      } else {
        currentNumber = Math.floor(Math.random() * 52) + 1;
      }
      numberDisplay.textContent = `Number: ${currentNumber}`;
      updatePressHistory('number');
    }

    function showBoth() {
      const numberDisplay = document.getElementById('number-display');

      const last3 = lastPresses.slice(-3);
      const sequenceIsCardNumberBoth = last3.length === 3 &&
                                      last3[0] === 'card' &&
                                      last3[1] === 'number' &&
                                      last3[2] === 'both';

      if (sequenceIsCardNumberBoth && currentNumber !== null) {
        currentCard = numberToCard(currentNumber);
        displayCard(currentCard);
        numberDisplay.textContent = `Number: ${currentNumber}`;
      } else {
        currentNumber = Math.floor(Math.random() * 52) + 1;

        let randomCard;
        do {
          const randomIndex = Math.floor(Math.random() * 52);
          randomCard = stack[randomIndex];
        } while (cardToNumber(randomCard) === currentNumber);

        currentCard = randomCard;
        displayCard(currentCard);
        numberDisplay.textContent = `Number: ${currentNumber}`;
      }

      updatePressHistory('both');
    }

    // Hide stack if visible
    function hideStackIfVisible() {
      const stackDiv = document.getElementById('stack-display');
      if (stackDiv.style.display === 'block') {
        stackDiv.style.display = 'none';
      }
    }

    // Handle button presses + hide stack if visible
    function buttonPressed(type) {
      hideStackIfVisible();
      if (type === 'card') showCard();
      else if (type === 'number') showNumber();
      else if (type === 'both') showBoth();
    }

    // Secret triple-click to show stack; single-click to hide if visible
    document.getElementById('title').addEventListener('click', () => {
      const stackDiv = document.getElementById('stack-display');

      if (stackDiv.style.display === 'block') {
        // Single click hides stack if visible
        stackDiv.style.display = 'none';
        clickCount = 0;
        if (clickTimer) {
          clearTimeout(clickTimer);
          clickTimer = null;
        }
        return;
      }

      // If stack not visible, count clicks for triple-click
      clickCount++;
      if (clickTimer) clearTimeout(clickTimer);
      clickTimer = setTimeout(() => {
        clickCount = 0;
      }, 1000);

      if (clickCount === 3) {
        clickCount = 0;
        let formattedStack = stack.map((card, i) => `${i + 1}: ${card}`).join('\n');
        stackDiv.textContent = formattedStack;
        stackDiv.style.display = 'block';
      }
    });

    window.onload = function() {
      currentNumber = Math.floor(Math.random() * 52) + 1;
      currentCard = numberToCard(currentNumber);
      displayCard(currentCard);
      document.getElementById('number-display').textContent = `Number: ${currentNumber}`;
    };
  </script>
</body>
</html>
