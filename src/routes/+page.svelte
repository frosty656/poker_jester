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
					flag: 'No Flag',
					toFlop: 2
				};
const positionList = ['BB', 'SB', 'B', 'CO', 'HJ', 'LJ', 'UTG2', 'UTG1', 'UTG'];
let positions = ['BB', 'SB', 'B', 'CO', 'HJ', 'LJ', 'UTG1', 'UTG'];
const actions = ['Fold', 'Call', 'Raise'];
const cards = ['A', 'K', 'Q', 'J', 'T', '9', '8', '7', '6', '5', '4', '3', '2'];
const suited = ['Suited', 'Offsuit'];
const playersList = [2, 3, 4, 5, 6, 7, 8, 9];
let players = ['0', 2, 3, 4, 5, 6, 7, 8]
const results = ['Lost Flop', 'Won Flop', 'Lost Turn', 'Won Turn', 'Lost River', 'Won River', 'Lost Showdown', 'Won Showdown']
const flags = ['Flag', 'No Flag']
const afterReview = ['Add Hand','Hand History']
let currentPosition = 2;
let handReview = -1;
let handHistory = [];
let gameHistory = [];

onMount(() => {
	for (let i = 0; i < localStorage.length; i++) {
		const key = localStorage.key(i);
		gameHistory.push(key);
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
		if (handReview === 1) {
			step = 6;
		}
		else {
			step = 3;
		}
	}

	function handlePlayers(p) {
		currentHand.playerCount = parseInt(p);
		document.querySelectorAll('.player-selection button').forEach((btn) => {
			btn.classList.toggle("active", btn.textContent === p);
		});

		players = ['0'];
		{console.log(players)}
		for (let i = 0; i < currentHand.playerCount - 1; i++) {
			console.log(players)
			players.push(playersList[i]);
		}
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

	function handleActionSelection(action) {
		if (action === 'Fold') {
			currentHand.action = 'Fold';
			currentHand.result = 'Fold';
			step = 6;
		}
		else {
			currentHand.action = action;
		}
	}

	function handleAction(p) {
		console.log(p);
		currentHand.toFlop = p;
		if (p === '0') {
			currentHand.result = 'Won Preflop';
			currentHand.notes = 'Won Preflop';
			step = 6;
		}
		else {
			currentHand.notes = p + ' to flop ';
			if (handReview === 1) {
				step = 6;
			}
			else {
				step = 5;
			}
		}
	}

	function handleResult(result) {
		currentHand.result = result;
		step = 6;
	}

	function handleAfterReview(a) {
		if (a === 'Add Hand') {
			currentHand.suited = 'Offsuit';
			currentHand.card1 = null;
			currentHand.card2 = null;
			currentHand.toCall = currentHand.BB;
			currentHand.notes = '';
			currentHand.flag = 'No Flag';
			currentHand.toFlop = 2;
			handReview = 0;
			goToCardSelection();
		}
		else {
			goToHistory();
		}
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
		if (handReview === 1) {
			step = 6;
		}
		else {
			step = 4;
		}
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
		handHistory = gameDetails.handHistory ?? [];
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
			flag: currentHand.flag,
			toFlop: currentHand.toFlop,
			player: session.player,
			venue: session.venue,
			date: session.date
		});
		const key = `${session.player} - ${session.venue} - ${session.date}`;
		const value = JSON.stringify({ handHistory });
		localStorage.setItem(key, value);
		currentHand.suited = 'Offsuit';
		currentHand.card1 = null;
		currentHand.card2 = null;
		currentHand.toCall = currentHand.BB;
		currentHand.notes = '';
		currentHand.flag = 'No Flag';
		currentHand.toFlop = 2;
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
		currentHand.toFlop = 2;
		session.date = `${year}-${month.toString().padStart(2, '0')}-${day.toString().padStart(2, '0')}`;
		handReview = 0;
		step = 1;
	}

	function goToPositionAndPlayer() {
		if (handReview === 0) {
			step = 2;
		}
	}

	function goToCardSelection() {
		if (handReview === 0) {
			step = 3;
		}
	}

	function goToAction() {
		if (!currentHand.card2) {
			step = 3;	
		}
		else {
		if (handReview === 0) {
			step = 4;
		}
		}
	}

	function goToResult() {
		if (handReview === 0) {
			step = 5;
		}
	}

	function goToNotes(h) {
		console.log(h.position);
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
		currentHand.toFlop = h.toFlop;
		session.player = h.player;
		session.venue = h.venue;
		session.date = h.date;
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
					<button class = "button fullWidth" on:click={() => loadGame(game)}>{game}</button>
					<button class = "button delete" on:click={() => deleteGame(game)}>Delete</button>
				</div>
			{/each}
			<div class="flexHigh"></div>
			<button class="button confirm" on:click={goToNewGame}>Add New Game</button>
	
<!-- PPPPPPP    LL           AA     YY    YY  EEEEEEEE  RRRRRRR  -->
<!-- PP    PP   LL         AA  AA    YY  YY   EE        RR    RR -->
<!-- PPPPPPPP   LL        AAAAAAAA    YYYY    EEEEEE    RRRRRRR  -->
<!-- PP         LL        AA    AA     YY     EE        RR   RR  -->
<!-- PP         LLLLLLLL  AA    AA     YY     EEEEEEEE  RR    RR -->
		{:else if step === 1}
			<h2>Player Information</h2>
			<label for="player">Player:</label>
			<input id = "player" class="input" type="text" bind:value={session.player} />
			<label for="venue">Venue:</label>
			<input id="venue" class="input" type="text" bind:value={session.venue} />
			<label for="date">Date:</label>
			<input id="date" class="input" type="date" bind:value={session.date} />
			<div class="flexHigh"></div>
			<button class="button confirm fullWidth" on:click={confirmGameInfo}>Confirm</button>
			
<!-- PPPPPPP     OOOOOO    SSSSSSS  IIIIIIII  TTTTTTTT  IIIIIIII   OOOOOO   NN    NN -->
<!-- PP    PP   OO    OO  SS           II        TT        II     OO    OO  NNN   NN -->
<!-- PPPPPPPP   OO    OO   SSSSSS      II        TT        II     OO    OO  NN NN NN -->
<!-- PP         OO    OO        SS     II        TT        II     OO    OO  NN  NNNN -->
<!-- PP          OOOOOO   SSSSSSS   IIIIIIII     TT     IIIIIIII   OOOOOO   NN    NN -->
		{:else if step === 2}
		console.log(p);
		console.log(currentHand.playerCount);
			<div class="entryDisplay">
				<span>
					{currentHand.playerCount ?? ' '} players | {currentHand.position ?? ' '}  |  
					{currentHand.SB}/{currentHand.BB}
				</span>
			</div>
			<div class="flexWide">
				<div class="flexHigh">
					<label for="player-selection">Players</label>
						{#each playersList as p}
							<button class={(p === currentHand.playerCount) ? "button selected grow" : "button grow"} on:click={() => handlePlayers(p)}>{p}</button>
						{/each}
				</div>
				<div class="flexHigh">
					<label for="position-selection">Position</label>
						{#each positions as pos}
								<button class = "button confirm grow" on:click={() => handlePosition(pos)}>
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
			<button class="button gotoPage" on:click={goToPositionAndPlayer}>{currentHand.position} | 
				{currentHand.playerCount} players  {currentHand.SB}/{currentHand.BB}</button>
			<div class="entryDisplay">
						Hand:
						{currentHand.card1 ?? ' '}
						{currentHand.card2 ?? ' '}
						{currentHand.card2 != undefined ? ((currentHand.suited === 'Suited') ? 's' : 'o') : ' '}
			</div>
			<div class="flexHigh">
				<div class="flexWide">
					{#each cards as card}
						<button class = "button wide3 grow" on:click={() => handleCardSelection(card)}>{card}</button>
					{/each}
					<button class = "button done grow" on:click={() => handleDone()}>DONE</button>
					<button class="button backspace grow" on:click={() => handleUndoLastCard()}>DEL</button>
				</div>
			</div>
			<div class="wide">
				{#each suited as s}
					<button class = "button confirm wide2" on:click={() => handleToggleSuited(s)}>{s}</button>
				{/each}
			</div>
		<!-- </div> -->
	
<!--    AA       CCCCCCC  TTTTTTTT  IIIIIIII   OOOOOO   NN    NN -->
<!--  AA  AA    CC           TT        II     OO    OO  NNN   NN -->
<!-- AAAAAAAA   CC           TT        II     OO    OO  NN NN NN -->
<!-- AA    AA   CC           TT        II     OO    OO  NN  NNNN -->
<!-- AA    AA    CCCCCCC     TT     IIIIIIII   OOOOOO   NN    NN -->
		{:else if step === 4}
					<button class="button gotoPage" on:click={goToPositionAndPlayer}>{currentHand.position} | 
						{currentHand.playerCount} players  {currentHand.SB}/{currentHand.BB}</button>
					<button class="button gotoPage" on:click={goToCardSelection}>
						{currentHand.card1 ?? ' '}
						{currentHand.card2 ?? ' '}
						{(currentHand.suited === 'Suited' ? 's' : 'o')}
					</button>
					<div class="entryDisplay">To call: {currentHand.toCall}, {currentHand.action}
						{currentHand.action === 'Raise' ? currentHand.raiseAmount : ' '}
					</div>

					<label>
						To call: 
						<input class="input" inputmode="numeric" on:focus="{event => event.target.select()}" bind:value={currentHand.toCall} />
					</label>
				<div class="flexHigh">
					{#each actions as action}
							<button class={(action === "Fold") ? "button confirm grow" : (action === currentHand.action) ? "button selected grow" : "button grow"} on:click={() => handleActionSelection(action)}>{action}</button>
					{/each}
				</div>
				<label>
					Raise amount: 
					<input class="input" inputmode="numeric" on:focus="{event => event.target.select()}" bind:value={currentHand.raiseAmount} />
				</label>
				<div class="wide">
					{#each players as p}
						<button class = "button confirm wide3" on:click={() => handleAction(p)}>{p}</button>
					{/each}
				</div>

	
<!-- RRRRRRR    EEEEEEEE   SSSSSSS  UU    UU  LL        TTTTTTTT -->						
<!-- RR    RR   EE        SS        UU    UU  LL           TT    -->						
<!-- RRRRRRR    EEEEEE     SSSSSS   UU    UU  LL           TT    -->						
<!-- RR   RR    EE              SS  UU    UU  LL           TT    -->						
<!-- RR    RR   EEEEEEEE  SSSSSSS    UUUUUU   LLLLLLLL     TT    -->						
		{:else if step === 5}
			<button class="button gotoPage" on:click={goToPositionAndPlayer}>{currentHand.position} | 
				{currentHand.playerCount} players  {currentHand.SB}/{currentHand.BB}</button>
			<button class="button gotoPage" on:click={goToCardSelection}>
				{currentHand.card1 ?? ' '}
				{currentHand.card2 ?? ' '}
				{(currentHand.suited === 'Suited' ? 's' : 'o')}
			</button>
			<button class="button gotoPage" on:click={goToAction} >{currentHand.toCall} to call, 
				{currentHand.action} {currentHand.action === 'Raise' ? 
				currentHand.raiseAmount : ' '}</button>
			<div class="flexWide">
				{#each results as result}
					<button class="confirm button wide2" on:click={() => handleResult(result)}>{result}</button>
				{/each}
			</div>
	
<!-- NN    NN    OOOOOO   TTTTTTTT  EEEEEEEE   SSSSSSS -->						
<!-- NNN   NN   OO    OO     TT     EE        SS       -->						
<!-- NN NN NN   OO    OO     TT     EEEEEE     SSSSSS  -->						
<!-- NN  NNNN   OO    OO     TT     EE              SS -->						
<!-- NN    NN    OOOOOO      TT     EEEEEEEE  SSSSSSS  -->						
		 {:else if step === 6}
			<button class="button gotoPage" on:click={goToPositionAndPlayer}>{currentHand.position} | 
				{currentHand.playerCount} players  {currentHand.SB}/{currentHand.BB}</button>
			<button class="button gotoPage" on:click={goToCardSelection}>
				{currentHand.card1 ?? ' '}
				{currentHand.card2 ?? ' '}
				{(currentHand.suited === 'Suited' ? 's' : 'o')}
			</button>

			{#if !(currentHand.action === 'Fold')}
			<button class="button gotoPage" on:click={goToAction} >{currentHand.toCall} to call, {currentHand.action} 
				{currentHand.action === 'Raise' ? currentHand.raiseAmount : ' '}</button>
			<button class="button gotoPage" on:click={goToResult} >{currentHand.result}</button>
			{:else}
				<button class="button gotoPage" on:click={goToAction}>{currentHand.action}</button>
			{/if}
			<div class="flexHigh">
				<!-- rows="5" cols="50"  -->
				<textarea id="notes" bind:value={currentHand.notes}></textarea>
			</div>
			{#if (handReview === 0)}
				<div class="wide">
					{#each flags as flag}
						<button class="confirm button wide2" on:click={() => handleFlag(flag)}>{flag}</button>
					{/each}
				</div>
			{:else}
				<div class="flexWide">
					{#each afterReview as a}
						<button class="confirm button wide2" on:click={() => handleAfterReview(a)}>{a}</button>
					{/each}
				</div>
			{/if}
		
<!-- HH    HH   IIIIIIII   SSSSSSS  TTTTTTTT   OOOOOO   RRRRRRR   YY    YY -->						
<!-- HH    HH      II     SS           TT     OO    OO  RR    RR   YY  YY  -->						
<!-- HHHHHHHH      II      SSSSSS      TT     OO    OO  RRRRRRR     YYYY   -->						
<!-- HH    HH      II           SS     TT     OO    OO  RR   RR      YY    -->						
<!-- HH    HH   IIIIIIII  SSSSSSS      TT      OOOOOO   RR    RR     YY    -->											
		{:else if step === 7}
			<div class="game-history">
				{#each handHistory as hand}
						<div class={(hand.flag === "Flag") ? "gotoFlaggedHand" : "gotoPage"}>
							<button on:click={goToNotes(hand)}>
								{hand.position} |
								{hand.card1} {hand.card2} {hand.suited} | {hand.toCall} to call, {hand.action} 
								{hand.action === 'Raise' ? hand.raiseAmount : ' '} | {hand.result}
							</button>
						</div>
				{/each}
			</div>
			<div class="confirm">
				<button class="confirm-button" on:click={goToSessions}>Back to Sessions</button>
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
		height: 97dvh;
		background: #D9D9D9;
		border-radius: 6px;
		display: flex;
		flex-direction: column;
		padding: 0.5%;
		row-gap: 1%;
	}



	.button {
		font-size: 20px;
		border: 3px solid #201b8a;
		border-radius: 6px;
		background-color: #f8f9fa;
		color: #007bff;
		padding: 2%;
		margin: 0.25%;
	}

	.selected {
		background-color: #007bff;
		color: #f8f9fa;
	}

	.confirm {
		background-color: #28a745;
		border-color: #023a0f;
		color: #000000;
	}

	.grow {
		flex-grow: 1;
	}

	.delete {
		background-color: #dc3545;
		border-color: #510d01;
		color: #ffffff;
		width: 20%;
	}

	.fullWidth {
		width: 100%;
	}

	.wide2 {
		width: 49%;
	}

	.wide3 {
		width: 32%;
	}

	.side-by-side {
		display: flex;
		gap: 1%;
	}

	.entryDisplay {
		font-size: 20px;
		border: 3px solid #7d4f06;
		border-radius: 4px;
		background-color: #ce9e52;
		text-align: center;
		padding: 2%;
	}

	.input {
		padding: 10px;
		font-size: 16px;
		border: 1px solid #ccc;
		border-radius: 4px;
		box-sizing: border-box;
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
		overflow-y: auto;
	}

	.gotoPage{
		background-color: #37cd64;
		border-color: #03511a;
		color: #000000;
		width: 100%;
	}

	.gotoFlaggedHand button{
		background-color: #af2020;
		border-color: #430606;
		color: #000000;
		font-size: 20px;
		border-radius: 6px;
		margin-bottom: 1%;
	}
	
/* PPPPPPP     OOOOOO    SSSSSSS  IIIIIIII  TTTTTTTT  IIIIIIII   OOOOOO   NN    NN */
/* PP    PP   OO    OO  SS           II        TT        II     OO    OO  NNN   NN */
/* PPPPPPPP   OO    OO   SSSSSS      II        TT        II     OO    OO  NN NN NN */
/* PP         OO    OO        SS     II        TT        II     OO    OO  NN  NNNN */
/* PP          OOOOOO   SSSSSSS   IIIIIIII     TT     IIIIIIII   OOOOOO   NN    NN */
	.flexHigh {
		display: flex;
		flex-direction: column;
		flex-grow: 1;
		justify-content: space-between;
		align-items: space-between;

	}

	.flexWide {
		display: flex;
		flex-grow: 1;
		flex-wrap: wrap;
		justify-content: space-between;
		align-items: space-between;
	}

	.wide {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-between;
		align-items: space-between;
	}

/* NN    NN    OOOOOO   TTTTTTTT  EEEEEEEE   SSSSSSS */						
/* NNN   NN   OO    OO     TT     EE        SS       */						
/* NN NN NN   OO    OO     TT     EEEEEE     SSSSSS  */						
/* NN  NNNN   OO    OO     TT     EE              SS */						
/* NN    NN    OOOOOO      TT     EEEEEEEE  SSSSSSS  */						

	textarea {
		flex-grow: 1;
		font-size: 20px;
		border: 1px solid #ccc;
		border-radius: 4px;
		background-color: #f8f9fa;
		margin: 0.5%;
		text-align: center;
	}

	.backspace{
		background-color: #dc3545;
		border-color: #510d01;
		color: #ffffff;
		width: 32%;
	}
	
	.done{
		background-color: #c79297;
		border-color: #280701;
		color: #440505;
		width: 32%;
	}
</style>