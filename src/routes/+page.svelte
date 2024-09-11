<script>
	
// JJJJJJJJ      AA     VV    VV     AA      SSSSSSS   CCCCCCC  RRRRRRR   IIIIIIII  PPPPPPP   TTTTTTTT
//    JJ       AA  AA    VV  VV    AA  AA   SS        CC        RR    RR     II     PP    PP     TT
//    JJ      AAAAAAAA   VV  VV   AAAAAAAA   SSSSSS   CC        RRRRRRR      II     PPPPPPPP     TT
// J  JJ      AA    AA     VV     AA    AA        SS  CC        RR   RR      II     PP           TT
//  JJJ       AA    AA     VV     AA    AA  SSSSSSS    CCCCCCC  RR    RR  IIIIIIII  PP           TT
import { onMount } from 'svelte';
let step = -1;
const now = new Date();
const year = now.getFullYear();
const month = now.getMonth() + 1; // JavaScript months are 0-indexed
const day = now.getDate();
let session = 	{	player: '',
					venue: '',
					date: `${year}-${month.toString().padStart(2, '0')}-${day.toString().padStart(2, '0')}`
				};
let currentHand = { suited: 'Offsuit', 
					card1: null, 
					card2: null, 
					position: 'B', 
					playerCount: 8,
					toCall: 2,
					SB: 1,
					BB: 2,
					action: '',
					result: '',
					raiseAmount: 6,
					notes: '' ,
					flag: 'No Flag'
				};
const positionList = ['BB', 'SB', 'B', 'CO', 'HJ', 'LJ', 'UTG', 'UTG1', 'UTG2'];
let positions = ['BB', 'SB', 'B', 'CO', 'HJ', 'LJ', 'UTG', 'UTG1'];
const actions = ['Fold', 'Call', 'Raise'];
const cards = ['A', 'K', 'Q', 'J', 'T', '9', '8', '7', '6', '5', '4', '3', '2'];
const suited = ['Suited', 'Offsuit'];
const players = [2, 3, 4, 5, 6, 7, 8, 9];
const results = ['Lost Flop', 'Won Flop', 'Lost Turn', 'Won Turn', 'Lost River', 'Won River', 'Lost Showdown', 'Won Showdown']
const flags = ['Flag', 'No Flag']
let currentPosition = 2;
let handReview = -1;
let handHistory = [];
let gameHistory = [];

onMount(() => {
	for (let i = 0; i < localStorage.length; i++) {
		const key = localStorage.key(i);
		gameHistory.push(key);
		// console.log(key);
	}
	step = 0;
});
	
// HH    HH      AA     NN    NN  DDDDDD    LL        EEEEEEEE 						
// HH    HH    AA  AA   NNN   NN  DD    DD  LL        EE       						
// HHHHHHHH   AAAAAAAA  NN NN NN  DD    DD  LL        EEEEEE   						
// HH    HH   AA    AA  NN  NNNN  DD    DD  LL        EE       						
// HH    HH   AA    AA  NN    NN  DDDDDD    LLLLLLLL  EEEEEEEE 						
	function handlePosition(pos) {
		currentHand.position = pos;
		document.querySelectorAll('.position-selection button').forEach((btn) => {
			btn.classList.toggle('active', btn.textContent === pos);
		});
		step = 3;
	}

	function handlePlayers(p) {
		currentHand.playerCount = parseInt(p);
		positions = [];
		for (let i = 0; i < Math.min(6, currentHand.playerCount); i++) {
			positions.push(positionList[i]);
		}
		if (currentHand.playerCount === 7) {
			positions.push('UTG');
		}
		if (currentHand.playerCount === 8) {
			positions.push('UTG1');
			positions.push('UTG');
		}
		if (currentHand.playerCount === 9) {
			positions.push('UTG2');
			positions.push('UTG1');
			positions.push('UTG');
		}
	}

	function handleCardSelection(card) {
		if (!currentHand.card1) {
			currentHand.card1 = card;
		} else if (!currentHand.card2) {
			currentHand.card2 = card;
		} else {
			alert('You may only select two cards');
		}
	}

	function handleAction(action) {
		currentHand.action = action;
		if (action === 'Fold') {
			currentHand.result = 'Fold';
			step = 6
		}
		else {
			step = 5;
		}
	}

	function handleResult(result) {
		// console.log(result);
		currentHand.result = result;
		step = 6;
	}

	function handleToggleSuited(s) {
		if (currentHand.card2){
			currentHand.suited = s;
			confirmCardSelection();
		}
	}

	function handleFlag(f) {
		currentHand.flag = f;
		currentPosition = positions.indexOf(currentHand.position);
		if (currentPosition === positions.length - 1) {
			currentPosition = 0;
		} 
		else {
			currentPosition = currentPosition + 1;
		}
		updateLocalStorage()
		currentHand.position = positions[currentPosition];
		step = 3;
	}

	function handleUndoLastCard() {
		if (currentHand.card2) {
			currentHand.card2 = null;
		} else if (currentHand.card1) {
			currentHand.card1 = null;
		}
	}

	function handleDone() {
		currentHand.suited = 'Offsuit';
		currentHand.card1 = null;
		currentHand.card2 = null;
		currentHand.notes = '';
		currentHand.flag = 'No Flag';
		session.player = '';
		session.venue = '';
		step = 0
	}

//  CCCCCCC    OOOOOO   NN    NN  FFFFFFFF  IIIIIIII  RRRRRRR  MM      MM  SSSSSSS 						
// CC         OO    OO  NNN   NN  FF           II     RR    RR MMM    MMM SS       						
// CC         OO    OO  NN NN NN  FFFFFF       II     RRRRRRR  MM MMMM MM  SSSSSS  						
// CC         OO    OO  NN  NNNN  FF           II     RR   RR  MM  MM  MM       SS 						
//  CCCCCCC    OOOOOO   NN    NN  FF        IIIIIIII  RR    RR MM  MM  MM SSSSSSS  						

	function confirmGameInfo() {
		const key = `${session.player} - ${session.venue} - ${session.date}`;
		const value = JSON.stringify({ session, currentHand });
		localStorage.setItem(key, value);
		gameHistory.push(key);
		step = 2;
	}

	function confirmCardSelection() {
		step = 4;
	}

//  SSSSSSS   TTTTTTTT   OOOOOO   RRRRRRR      AA      GGGGGG   EEEEEEEE 						
// SS            TT     OO    OO  RR    RR   AA  AA   GG        EE       						
//  SSSSSS       TT     OO    OO  RRRRRRR   AAAAAAAA  GG   GGG  EEEEEE   						
//       SS      TT     OO    OO  RR   RR   AA    AA  GG     G  EE       						
// SSSSSSS       TT      OOOOOO   RR    RR  AA    AA   GGGGGG   EEEEEEEE 						

	function loadGame(key) {
		console.clear()
		const value = localStorage.getItem(key);
		const gameDetails = JSON.parse(value);
		// console.log(value);
		// console.log(gameDetails);

		// session.player = gameDetails.player;
		// session.venue = gameDetails.venue;
		// session.date = gameDetails.date;
		handHistory = gameDetails.handHistory ?? [];
		// currentHand.position = handHistory['position'];
		// currentHand.playerCount = handHistory.playerCount;
		// currentHand.card1 = handHistory.card1;
		// currentHand.card2 = handHistory.card2;
		// currentHand.suited = handHistory.suited;
		// currentHand.toCall = handHistory.toCall;
		// currentHand.action = handHistory.action;
		// currentHand.raiseAmount = handHistory.raiseAmount;
		// currentHand.result = handHistory.result;
		// currentHand.notes = handHistory.notes;
		// currentHand.SB = handHistory.SB;
		// currentHand.BB = handHistory.BB;
		// currentHand.flag = handHistory.flag;
		handReview = 1;
		step = 7;
	}

	function deleteGame(key) {
		if (!confirm('Are you sure you want to delete this game?')) {
			return;
		}
		localStorage.removeItem(key);
		gameHistory = gameHistory.filter((game) => game !== key);
	}

	function updateLocalStorage() {
		handHistory.push({
			position: currentHand.position,
			playerCount: currentHand.playerCount,
			card1: currentHand.card1,
			card2: currentHand.card2,
			suited: currentHand.suited,
			toCall: currentHand.toCall,
			action: currentHand.action,
			raiseAmount: currentHand.raiseAmount,
			result: currentHand.result,
			notes: currentHand.notes,
			SB: currentHand.SB,
			BB: currentHand.BB,
			flag: currentHand.flag
		});
		// sessionValues.push({
		// 	player: session.player,
		// 	venue: session.venue,
		// 	date: session.date
		// })
		const key = `${session.player} - ${session.venue} - ${session.date}`;
		// const value = JSON.stringify({ session, handHistory });
		const value = JSON.stringify({ handHistory });
		localStorage.setItem(key, value);
		currentHand.suited = 'Offsuit';
		currentHand.card1 = null;
		currentHand.card2 = null;
		currentHand.notes = '';
		currentHand.flag = 'No Flag';
	}

// JJJJJJJJ   UU    UU MM      MM PPPPPPP             PPPPPPP      AA      GGGGGG   EEEEEEEE
//    JJ      UU    UU MMM    MMM PP    PP            PP    PP   AA  AA   GG        EE
//    JJ      UU    UU MM MMMM MM PPPPPPPP            PPPPPPPP  AAAAAAAA  GG   GGG  EEEEEE
// J  JJ      UU    UU MM  MM  MM PP                  PP        AA    AA  GG     G  EE
//  JJJ        UUUUUU  MM  MM  MM PP                  PP        AA    AA   GGGGGG   EEEEEEEE

	function goToSessions() {
		handReview = 1;
		step = 0;
	}

	function goToNewGame() {
		currentHand.position = 'B';
		currentHand.playerCount = 8;
		currentHand.card1 = null;
		currentHand.card2 = null;
		currentHand.suited = 'Offsuit';
		currentHand.toCall = 2;
		currentHand.action = '';
		currentHand.raiseAmount = 6;
		currentHand.result = '';
		currentHand.notes = '';
		currentHand.SB = 1;
		currentHand.BB = 2;
		currentHand.flag = 'No Flag';
		session.date = `${year}-${month.toString().padStart(2, '0')}-${day.toString().padStart(2, '0')}`;
		handReview = 0;
		step = 1;
	}

	function goToPositionAndPlayer() {
		step = 2;
	}

	function goToCardSelection() {
		step = 3;
	}

	function goToAction() {
		if (!currentHand.card2) {
			step = 3;	
		}
		else {
		step = 2;}
	}

	function goToResult() {
		step = 5;
	}

	function goToNotes(h) {
		console.log(h.position)
		// currentHand = hand;
		currentHand.position = h.position;
		currentHand.playerCount = h.playerCount;
		currentHand.card1 = h.card1;
		currentHand.card2 = h.card2;
		currentHand.suited = h.suited;
		currentHand.toCall = h.toCall;
		currentHand.action = h.action;
		currentHand.raiseAmount = h.raiseAmount;
		currentHand.result = h.result;
		currentHand.notes = h.notes;
		currentHand.SB = h.SB;
		currentHand.BB = h.BB;
		currentHand.flag = h.flag;
		step = 6;
	}

	function goToHistory() {
		step = 7;
	}
</script>
	
	<!-- HH    HH   TTTTTTTT MM      MM LL       -->
	<!-- HH    HH      TT    MMM    MMM LL       -->
	<!-- HHHHHHHH      TT    MM MMMM MM LL       -->
	<!-- HH    HH      TT    MM  MM  MM LL       -->
	<!-- HH    HH      TT    MM  MM  MM LLLLLLLL -->
	
	<!--  SSSSSSS   EEEEEEEE   SSSSSSS   SSSSSSS  IIIIIIII   OOOOOO   NN    NN -->						
	<!-- SS         EE        SS        SS           II     OO    OO  NNN   NN -->						
	<!--  SSSSSS    EEEEEE     SSSSSS    SSSSSS      II     OO    OO  NN NN NN -->						
	<!--       SS   EE              SS        SS     II     OO    OO  NN  NNNN -->						
	<!-- SSSSSSS    EEEEEEEE  SSSSSSS   SSSSSSS   IIIIIIII   OOOOOO   NN    NN -->						
	<div class="container">
		{#if step === 0}
			{#each gameHistory as game}
				<div class="side-by-side">
					<button on:click={() => loadGame(game)}>
						{game}
					</button>
					<div class="delete-button">
						<button on:click={() => deleteGame(game)}>
							Delete
						</button>
					</div>
				</div>
			{/each}
			<div class="flexfill"></div>
			<div class="confirm">
				<button class="confirm-button" on:click={goToNewGame}>Add New Game</button>
			</div>
	
	<!-- PPPPPPP    LL           AA     YY    YY  EEEEEEEE  RRRRRRR  -->
	<!-- PP    PP   LL         AA  AA    YY  YY   EE        RR    RR -->
	<!-- PPPPPPPP   LL        AAAAAAAA    YYYY    EEEEEE    RRRRRRR  -->
	<!-- PP         LL        AA    AA     YY     EE        RR   RR  -->
	<!-- PP         LLLLLLLL  AA    AA     YY     EEEEEEEE  RR    RR -->
		{:else if step === 1}
			<div class="player-info">
				<h2>Player Information</h2>
				<label for="player">Player:</label>
				<input id="player" type="text" bind:value={session.player} />
				<label for="venue">Venue:</label>
				<input id="venue" type="text" bind:value={session.venue} />
				<label for="date">Date:</label>
				<input id="date" type="date" bind:value={session.date} />
			</div>
			<div class="flexfill"></div>
			<div class="confirm">
				<button class="confirm-button" on:click={confirmGameInfo}>Confirm</button>
			</div>
			
	
	<!-- PPPPPPP     OOOOOO    SSSSSSS  IIIIIIII  TTTTTTTT  IIIIIIII   OOOOOO   NN    NN -->
	<!-- PP    PP   OO    OO  SS           II        TT        II     OO    OO  NNN   NN -->
	<!-- PPPPPPPP   OO    OO   SSSSSS      II        TT        II     OO    OO  NN NN NN -->
	<!-- PP         OO    OO        SS     II        TT        II     OO    OO  NN  NNNN -->
	<!-- PP          OOOOOO   SSSSSSS   IIIIIIII     TT     IIIIIIII   OOOOOO   NN    NN -->
		{:else if step === 2}
			<div class="entry-display">
				<span>
					{currentHand.playerCount ?? ' '} players | {currentHand.position ?? ' '}  |  
					{currentHand.SB}/{currentHand.BB}
				</span>
			</div>
			<div class="player-position">
				<div class="player-selection">
					<label for="player-selection">Players</label>
						{#each players as p}
								<button on:click={() => handlePlayers(p)}>
									{p}
								</button>
						{/each}
				</div>
				
				<div class="position-selection">
					<label for="position-selection">Position</label>
						{#each positions as pos}
								<!-- <button class={pos === currentHand.position ? 'active' : ''} on:click={() => handlePosition(pos)}> -->
								<button on:click={() => handlePosition(pos)}>
									{pos}
								</button>
						{/each}
				</div>
			</div>
			<div class="Blinds">
				<div class="side-by-side">
					<label for="Blinds">SB:</label>
					<input id="SB" type="text" bind:value={currentHand.SB} />
					<label for="Blinds"> BB:</label>
					<input id="BB" type="text" bind:value={currentHand.BB} />
				</div>
			</div>
	
	<!--  CCCCCCC      AA     RRRRRRR   DDDDDD     SSSSSSS -->
	<!-- CC          AA  AA   RR    RR  DD    DD  SS       -->
	<!-- CC         AAAAAAAA  RRRRRRR   DD    DD   SSSSSS  -->
	<!-- CC         AA    AA  RR   RR   DD    DD        SS -->
	<!--  CCCCCCC   AA    AA  RR    RR  DDDDDD    SSSSSSS  -->
		{:else if step === 3}
			<div class="gotoPage-button"> 
				<button on:click={goToPositionAndPlayer}>{currentHand.position} | 
					{currentHand.playerCount} players  {currentHand.SB}/{currentHand.BB}</button>
			</div>
			<div class="entry-display">
				<div class="side-by-side-grid">
					<div>Hand:
						{currentHand.card1 ?? ' '}
						{currentHand.card2 ?? ' '}
						{currentHand.card2 != undefined ? ((currentHand.suited === 'Suited') ? 's' : 'o') : ' '}

					</div>
				</div>
			</div>
			<div class="cards">
				<div class="card-grid">
					<div class="card">
						{#each cards as card}
							<button on:click={() => handleCardSelection(card)}>{card}</button>
						{/each}
						<div class="done-button">
							<button on:click={() => handleDone()}>DONE</button>
						</div>
						<div class="backspace-button">
							<button on:click={() => handleUndoLastCard()}>DEL</button>
						</div>
					</div>
				</div>
			</div>
			<div class="confirm">
			<div class="suited-grid">
				{#each suited as s}
					<button on:click={() => handleToggleSuited(s)}>{s}</button>
				{/each}
			</div>
		</div>
				<!-- <div>
					<button class="" on:click={() => {step = 7}}>View History</button> -->
				<!-- </div> -->
	
	<!--    AA       CCCCCCC  TTTTTTTT  IIIIIIII   OOOOOO   NN    NN -->
	<!--  AA  AA    CC           TT        II     OO    OO  NNN   NN -->
	<!-- AAAAAAAA   CC           TT        II     OO    OO  NN NN NN -->
	<!-- AA    AA   CC           TT        II     OO    OO  NN  NNNN -->
	<!-- AA    AA    CCCCCCC     TT     IIIIIIII   OOOOOO   NN    NN -->
		{:else if step === 4}
				<div class="gotoPage-button"> 
					<button on:click={goToPositionAndPlayer}>{currentHand.position} | 
						{currentHand.playerCount} players  {currentHand.SB}/{currentHand.BB}</button>
					<button on:click={goToCardSelection}>
						{currentHand.card1 ?? ' '}
						{currentHand.card2 ?? ' '}
						{(currentHand.suited === 'Suited' ? 's' : 'o')}
					</button>
				</div>
				<div class="entry-display">
					<span
						>To call: {currentHand.toCall}, {currentHand.action}
						{currentHand.action === 'Raise' ? currentHand.raiseAmount : ' '}
					</span>
				</div>
				<div class="side-by-side">
						<label for="toCall">To call:</label>
						<input id="toCall" type="text" bind:value={currentHand.toCall} />
				</div>
				<div class="action">
					{#each actions as action}
						<button on:click={() => handleAction(action)}>{action}</button>
					{/each}
				</div>
				<div class="side-by-side">
					<label for="raise-amount">Raise amount:</label>
					<input id="raise-amount" type="text" bind:value={currentHand.raiseAmount} />
				</div>
	
	<!-- RRRRRRR    EEEEEEEE   SSSSSSS  UU    UU  LL        TTTTTTTT -->						
	<!-- RR    RR   EE        SS        UU    UU  LL           TT    -->						
	<!-- RRRRRRR    EEEEEE     SSSSSS   UU    UU  LL           TT    -->						
	<!-- RR   RR    EE              SS  UU    UU  LL           TT    -->						
	<!-- RR    RR   EEEEEEEE  SSSSSSS    UUUUUU   LLLLLLLL     TT    -->						
		{:else if step === 5}
			<div class="gotoPage-button"> 
				<button on:click={goToPositionAndPlayer}>{currentHand.position} | 
					{currentHand.playerCount} players  {currentHand.SB}/{currentHand.BB}</button>
				<button on:click={goToCardSelection}>
					{currentHand.card1 ?? ' '}
					{currentHand.card2 ?? ' '}
					{(currentHand.suited === 'Suited' ? 's' : 'o')}
				</button>
				<button on:click={goToAction} >{currentHand.toCall} to call, 
					{currentHand.action} {currentHand.action === 'Raise' ? 
					currentHand.raiseAmount : ' '}</button>
			</div>
			<div class="result">
			<div class="result-grid">
					{#each results as result}
						<button class="confirm-button" on:click={() => handleResult(result)}>{result}</button>
					{/each}
				</div>
			</div>
	
	<!-- NN    NN    OOOOOO   TTTTTTTT  EEEEEEEE   SSSSSSS -->						
	<!-- NNN   NN   OO    OO     TT     EE        SS       -->						
	<!-- NN NN NN   OO    OO     TT     EEEEEE     SSSSSS  -->						
	<!-- NN  NNNN   OO    OO     TT     EE              SS -->						
	<!-- NN    NN    OOOOOO      TT     EEEEEEEE  SSSSSSS  -->						
		 {:else if step === 6}
			<div class="gotoPage-button"> 
				<button on:click={goToPositionAndPlayer}>{currentHand.position} | 
					{currentHand.playerCount} players  {currentHand.SB}/{currentHand.BB}</button>
				<button on:click={goToCardSelection}>
					{currentHand.card1 ?? ' '}
					{currentHand.card2 ?? ' '}
					{(currentHand.suited === 'Suited' ? 's' : 'o')}
				</button>
				{#if !(currentHand.action === 'Fold')}
					<button on:click={goToAction} >{currentHand.toCall} to call, {currentHand.action} 
						{currentHand.action === 'Raise' ? currentHand.raiseAmount : ' '}</button>
					<button on:click={goToResult} >{currentHand.result}</button>
				{:else}
					<button on:click={goToAction}>{currentHand.action}</button>
				{/if}
			</div>
			<div class="notes">
				<textarea rows="5" cols="50" id="notes" bind:value={currentHand.notes}></textarea>
			</div>
			{#if (handReview === 0)}
				<div class="flag">
					<div class="flag-grid">
						{#each flags as flag}
							<button on:click={() => handleFlag(flag)}>{flag}</button>
						{/each}
					</div>
				</div>
			{:else}
				<div class="confirm">
					<button class="confirm-button" on:click={goToHistory()}>Back to Hand History</button>
				</div>
			{/if}
		
<!-- HH    HH   IIIIIIII   SSSSSSS  TTTTTTTT   OOOOOO   RRRRRRR   YY    YY -->						
<!-- HH    HH      II     SS           TT     OO    OO  RR    RR   YY  YY  -->						
<!-- HHHHHHHH      II      SSSSSS      TT     OO    OO  RRRRRRR     YYYY   -->						
<!-- HH    HH      II           SS     TT     OO    OO  RR   RR      YY    -->						
<!-- HH    HH   IIIIIIII  SSSSSSS      TT      OOOOOO   RR    RR     YY    -->											
		{:else if step === 7}
		<div>
			{#each handHistory as hand}
				<div class="gotoPage-button">
					<button on:click={goToNotes(hand)}>
						{hand.position} |
						{hand.card1} {hand.card2} {hand.suited} | {hand.toCall} to call, {hand.action} 
						{hand.action === 'Raise' ? hand.raiseAmount : ' '} | {hand.result}
					</button>
				</div>
			{/each}
			<div class="confirm">
				<button class="confirm-button" on:click={goToSessions}>Back to Sessions</button>
			</div>
		</div>

		{/if}
	</div>

	<style>
	/*  CCCCCCC    SSSSSSS   SSSSSSS */
	/* CC         SS        SS       */
	/* CC          SSSSSS    SSSSSS  */
	/* CC               SS        SS */
	/*  CCCCCCC   SSSSSSS   SSSSSSS  */
	
		.container {
/*			height: 600px;
			width: 450px; */
			height: 100vh;
			background: #D9D9D9;
			border-radius: 8px;
			box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
			display: flex;
			flex-direction: column;
			padding: 0.5%;
			row-gap: 1%;
		}
	
		.container button {
			font-size: 20px;
			border: 3px solid #201b8a;
			border-radius: 8px;
			background-color: #f8f9fa;
			color: #007bff;
			padding: 2%;
			width: 100%; 
		}

		.confirm button {
			background-color: #28a745;
			border-color: #023a0f;
			color: #000000;
			
		}
		.delete-button button {
			background-color: #dc3545;
			border-color: #510d01;
			color: #ffffff;
			height: 100%;
		}

		.side-by-side {
			display: flex;
			gap: 1%;
		}

		.flexfill {
			flex-grow: 1;
		}

		.entry-display {
			font-size: 20px;
			border: 3px solid #7d4f06;
			border-radius: 4px;
			background-color: #ce9e52;
			text-align: center;
			padding: 2%;
		}

		.container input {
			padding: 10px;
			font-size: 16px;
			border: 1px solid #ccc;
			border-radius: 4px;
			box-sizing: border-box;
			width: 100%;
		}
	

		.container label {
			font-weight: bold;
			margin-bottom: 1%;
			white-space: nowrap;
			display: flex;
			align-items: center;
		}


		.game-history {
			flex-grow: 1;
		}
	
		.side-by-side-grid {
			display: grid;
			grid-template-columns: repeat(2, auto);
			justify-content: space-between;
			flex-grow: 1;
		}
	
		.gotoPage-button button{
			background-color: #37cd64;
			border-color: #03511a;
			color: #000000;
			font-size: 20px;
			border-radius: 4px;
			margin-bottom: 1%;
		}
	
/* PPPPPPP     OOOOOO    SSSSSSS  IIIIIIII  TTTTTTTT  IIIIIIII   OOOOOO   NN    NN */
/* PP    PP   OO    OO  SS           II        TT        II     OO    OO  NNN   NN */
/* PPPPPPPP   OO    OO   SSSSSS      II        TT        II     OO    OO  NN NN NN */
/* PP         OO    OO        SS     II        TT        II     OO    OO  NN  NNNN */
/* PP          OOOOOO   SSSSSSS   IIIIIIII     TT     IIIIIIII   OOOOOO   NN    NN */
		.player-position {
			display: flex;
			flex-grow: 1;
		}

		.player-selection {
			display: flex;
			flex-direction: column;
			width: 50%;
		}
		.player-selection button{
			width: 100%;
			margin:1%;
			flex-grow: 1;
		}

		.position-selection {
			display: flex;
			flex-direction: column;
			width: 100%;
		}
		.position-selection button{
			width: 100%;
			margin:1%;
			flex-grow: 1;
			background-color: #28a745;
			border-color: #023a0f;
			color: #000000;
		}

/*  CCCCCCC      AA     RRRRRRR   DDDDDD     SSSSSSS */
/* CC          AA  AA   RR    RR  DD    DD  SS       */
/* CC         AAAAAAAA  RRRRRRR   DD    DD   SSSSSS  */
/* CC         AA    AA  RR   RR   DD    DD        SS */
/*  CCCCCCC   AA    AA  RR    RR  DDDDDD    SSSSSSS  */
		.cards {
			display: flex;
			flex-grow: 1;
		}

		.card-grid {
			display: flex;
			flex-grow: 1;
			width: 100%;
		}

		.card {
			display: flex;
			flex-wrap: wrap;
			flex-grow: 1;
		}

		.card button{
			margin:1%;
			flex-grow: 1;
			width: 30%;
		}

		.suited-grid {
			flex-grow: 1;
			display: flex;
			flex-wrap: wrap;
		}

		.suited-grid button{
			margin:1%;
			flex-grow: 1;
			width: 45%;
		}
	
/*    AA       CCCCCCC  TTTTTTTT  IIIIIIII   OOOOOO   NN    NN */						
/*  AA  AA    CC           TT        II     OO    OO  NNN   NN */						
/* AAAAAAAA   CC           TT        II     OO    OO  NN NN NN */						
/* AA    AA   CC           TT        II     OO    OO  NN  NNNN */						
/* AA    AA    CCCCCCC     TT     IIIIIIII   OOOOOO   NN    NN */						

		.action {
			flex-grow: 1;
			display: flex;
			flex-direction: column;
		}

		.action button {
			flex-grow: 1;
			margin: 1%;
			background-color: #28a745;
			border-color: #023a0f;
			color: #000000;
		}

/* RRRRRRR    EEEEEEEE   SSSSSSS  UU    UU  LL        TTTTTTTT */						
/* RR    RR   EE        SS        UU    UU  LL           TT    */						
/* RRRRRRR    EEEEEE     SSSSSS   UU    UU  LL           TT    */						
/* RR   RR    EE              SS  UU    UU  LL           TT    */						
/* RR    RR   EEEEEEEE  SSSSSSS    UUUUUU   LLLLLLLL     TT    */						

		.result {
			flex-grow: 1;
			display: flex;
			flex-direction: column;
		}

		.result-grid {
			flex-grow: 1;
			display: flex;
			flex-wrap: wrap;
		}
	
		.result button {
			margin: 1%;
			background-color: #28a745;
			border-color: #023a0f;
			color: #000000;
			width: 48%;
		}

/* NN    NN    OOOOOO   TTTTTTTT  EEEEEEEE   SSSSSSS */						
/* NNN   NN   OO    OO     TT     EE        SS       */						
/* NN NN NN   OO    OO     TT     EEEEEE     SSSSSS  */						
/* NN  NNNN   OO    OO     TT     EE              SS */						
/* NN    NN    OOOOOO      TT     EEEEEEEE  SSSSSSS  */						
		.notes {
			flex-grow: 1;
			display: flex;
			flex-direction: column;
		}

		textarea {
			flex-grow: 1;
			font-size: 20px;
			border: 1px solid #ccc;
			border-radius: 4px;
			background-color: #f8f9fa;
			width: 100%;
			text-align: center;
		}
	
		.flag-grid {
			/* flex-grow: 1; */
			display: flex;
			/* flex-wrap: wrap; */
		}
	
		.flag button {
			flex: 1;
			margin: 1%;
			background-color: #28a745;
			border-color: #023a0f;
			color: #000000;
			width: 48%;
		}

	.backspace-button{
			margin:1%;
			flex-grow: 1;
			width: 30%;
		}

	.backspace-button button{
		background-color: #dc3545;
		border-color: #510d01;
		color: #ffffff;
		margin:0%;
		flex-grow: 1;
		width: 100%;
		height: 100%;
	}
	
	.done-button{
			margin:1%;
			flex-grow: 1;
			width: 30%;
		}

	.done-button button{
		background-color: #c79297;
		border-color: #280701;
		color: #440505;
		margin:0%;
		flex-grow: 1;
		width: 100%;
		height: 100%;
	}
	

	.backspace-icon {
		background-color: #ce9e52;
		color: #c60d0d;
		font-size: 24px;
	}

	</style>