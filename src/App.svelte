<script>
	let faceCards = [
		'king',
		'queen',
		'jack'
	];

	const hand = {
		cards: [],
		drewHand: false,
		currentHandValue: '',
		drawHand: () => {
			fetch(`https://deckofcardsapi.com/api/deck/${deck.redraw ? 'new' : deck.id}/draw/?count=2`)
				.then((res)=>{
					res.json()
						.then((json)=>{
							if(json.remaining = 0){
								deck.redraw = true;
							}
							deck.id = json.deck_id;
							hand.cards = hand.cards.concat(json.cards);
							hand.drewHand = true;
							hand.calculateValue();
							console.log(json);
					});
				});
		},
		drawCard: () => {
			if (!hand.drewHand) return;
			fetch(`https://deckofcardsapi.com/api/deck/${deck.id}/draw/?count=1`)
				.then((res)=>{
					res.json()
						.then((json)=>{
							if(json.remaining = 0){
								deck.redraw = true;
							}
							hand.cards = hand.cards.concat(json.cards);
							hand.calculateValue();
							console.log(hand.cards);
					});
				});
		},
		calculateValue: () => {
			let currentValues = [];
			for (let i = 0; i < hand.cards.length; i++) {
				const element = hand.cards[i];
				if(faceCards.includes(element.value.toLowerCase())){
					currentValues.push(10);
				} else if(element.value === 'ACE') {
					currentValues.push(1);
				} else {
					currentValues.push(parseInt(element.value));
				}
			}
			hand.currentHandValue = currentValues.reduce((a, b) => a + b, 0);
			if(hand.currentHandValue >= 21){
				game.gameOver = true;
				if(parseInt(hand.currentHandValue) == 21){
					console.log('You got 21! Congrats!');
				} else {
					console.log('Mission failed. We\'ll get \'em next time.');
				}
			}
		},
		discard: () => {hand.cards = []}
	};

	const player = {
		playerName: '',
		playerId: '',
		hand: hand,
		getName: () => {
			console.log(player.playerName);
		}
	};

	const deck = {
		id: 'new',
		player: player,
		cards: '',
		redraw: false,
		shuffle: () => {
			fetch(`https://deckofcardsapi.com/api/deck/${deck.id ? deck.id : 'new'}/shuffle/?deck_count=1`)
				.then((res)=>{
					res.json()
						.then((json)=>{
							deck.id = json.deck_id;
							console.log(json);	
					});
				});
		}
	};

	const game = {
		deck: deck,
		players: [player],
		gameOver: false,
		reset: () => {
			hand.cards = [];
			game.gameOver = false;
			hand.drewHand = false;
			hand.currentHandValue = '';
		}
	};

</script>

<style>
	button{
		background: #6545a4;
		color: #ffffff;
		border: none;
		border-radius: 5px;
		padding: 10px 15px;
		cursor: pointer;
	}
	#shuffle-deck{
		background: #e7daff;
		color: #6545a4;
	}
	.buttons{
		display: flex;
    	justify-content: center;
	}
	.buttons button:nth-child(1){
		margin-right: 1em;
	}
	#value{
		text-align: center;
	}
	.cards{
		display: flex;
    	justify-content: space-evenly;
	}
</style>

<div class="buttons">
	<button id="shuffle-deck" on:click={deck.shuffle}>Shuffle Deck</button>
	<button hidden={hand.drewHand} on:click={hand.drawHand}>Draw Hand</button>
	<button hidden={!hand.drewHand} on:click={hand.drawCard}>Draw Card</button>
</div>
<br>
<p id="value" hidden={game.gameOver}>{hand.currentHandValue}</p>
<br>
<p hidden={!game.gameOver}>Game Over</p>
<br>
<button hidden={!game.gameOver} on:click={game.reset}>Reset</button>
<div class="cards">
	{#each hand.cards as { image }, i}
		<img hidden={game.gameOver} src="{image}" alt="">
	{/each}
</div>

