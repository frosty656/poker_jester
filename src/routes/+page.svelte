<script>
	// @ts-nocheck
	
		// JJJJJJJJ      AA     VV    VV     AA      SSSSSSS   CCCCCCC  RRRRRRR   IIIIIIII  PPPPPPP   TTTTTTTT
		//    JJ       AA  AA    VV  VV    AA  AA   SS        CC        RR    RR     II     PP    PP     TT
		//    JJ      AAAAAAAA   VV  VV   AAAAAAAA   SSSSSS   CC        RRRRRRR      II     PPPPPPPP     TT
		// J  JJ      AA    AA     VV     AA    AA        SS  CC        RR   RR      II     PP           TT
		//  JJJ       AA    AA     VV     AA    AA  SSSSSSS    CCCCCCC  RR    RR  IIIIIIII  PP           TT
		import { onMount } from 'svelte';
		let step = -1;
		let player = '';
		let venue = '';
		let date = '';
		let currentHand = { suited: 'Offsuit', 
							card1: null, 
							card2: null, 
							position: 'B', 
							playerCount: 8,
							toCall: 2,
							SB: 1,
							BB: 2,
							action: 'Fold',
							result: 'Lost Flop',
							raiseAmount: 6 };
		const positionList = ['BB', 'SB', 'B', 'CO', 'HJ', 'LJ', 'UTG', 'UTG1', 'UTG2'];
		let cp = 2;
		let positions = ['BB', 'SB', 'B', 'CO', 'HJ', 'LJ', 'UTG', 'UTG1'];
		let actions = ['Fold', 'Call', 'Raise'];
		const cards = ['A', 'K', 'Q', 'J', 'T', '9', '8', '7', '6', '5', '4', '3', '2'];
		const suited = ['Suited', 'Offsuit'];
		const players = [2, 3, 4, 5, 6, 7, 8, 9];
		const results = ['Lost Flop', 'Won Flop', 'Lost Turn', 'Won Turn', 'Lost River', 'Won River', 'Lost Showdown', 'Won Showdown']
		let notes = '';
		let handHistory = [];
		let gameHistory = [];
	
		onMount(() => {
			const now = new Date();
			const year = now.getFullYear();
			const month = now.getMonth() + 1; // JavaScript months are 0-indexed
			const day = now.getDate();
			date = `${year}-${month.toString().padStart(2, '0')}-${day.toString().padStart(2, '0')}`;
			for (let i = 0; i < localStorage.length; i++) {
				const key = localStorage.key(i);
				gameHistory.push(key);
				console.log(key);
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
		}
	
		function handleResult(result) {
			currentHand.result = result;
			step = 6;
		}
	
		function handleToggleSuited(s) {
			currentHand.suited = s;
			confirmCardSelection();
		}
	
		function handleUndoLastCard() {
			if (currentHand.card2) {
				currentHand.card2 = null;
			} else if (currentHand.card1) {
				currentHand.card1 = null;
			}
		}
	
	//  CCCCCCC    OOOOOO   NN    NN  FFFFFFFF  IIIIIIII  RRRRRRR  MM      MM  SSSSSSS 						
	// CC         OO    OO  NNN   NN  FF           II     RR    RR MMM    MMM SS       						
	// CC         OO    OO  NN NN NN  FFFFFF       II     RRRRRRR  MM MMMM MM  SSSSSS  						
	// CC         OO    OO  NN  NNNN  FF           II     RR   RR  MM  MM  MM       SS 						
	//  CCCCCCC    OOOOOO   NN    NN  FF        IIIIIIII  RR    RR MM  MM  MM SSSSSSS  						
	
		function confirmGameInfo() {
			const key = `${player} - ${venue} - ${date}`;
			const value = JSON.stringify({ player, venue, date, handHistory });
			localStorage.setItem(key, value);
			gameHistory.push(key);
			step = 2;
		}
	
		function confirmPlayerPosition() {
			step = 3;
		}
	
		function confirmCardSelection() {
			step = 4;
		}
	
		function confirmAction() {
			step = 5;
		}
	
	
	// NN    NN    OOOOOO   TTTTTTTT  EEEEEEEE   SSSSSSS 						
	// NNN   NN   OO    OO     TT     EE        SS       						
	// NN NN NN   OO    OO     TT     EEEEEE     SSSSSS  						
	// NN  NNNN   OO    OO     TT     EE              SS 						
	// NN    NN    OOOOOO      TT     EEEEEEEE  SSSSSSS  						
	
		function addHand(notes) {
			currentHand.notes = notes;
			cp = positions.indexOf(currentHand.position);
			if (cp > positions.length) 
			{cp = 1;} 
			else {cp=cp+1;}
			updateLocalStorage()
			position = positions(i);
			step = 3;
		}
	
	//  SSSSSSS   TTTTTTTT   OOOOOO   RRRRRRR      AA      GGGGGG   EEEEEEEE 						
	// SS            TT     OO    OO  RR    RR   AA  AA   GG        EE       						
	//  SSSSSS       TT     OO    OO  RRRRRRR   AAAAAAAA  GG   GGG  EEEEEE   						
	//       SS      TT     OO    OO  RR   RR   AA    AA  GG     G  EE       						
	// SSSSSSS       TT      OOOOOO   RR    RR  AA    AA   GGGGGG   EEEEEEEE 						
	
		function loadGame(key) {
			const value = localStorage.getItem(key);
			const gameDetails = JSON.parse(value);
			console.log(gameDetails);
			player = gameDetails.player;
			venue = gameDetails.venue;
			date = gameDetails.date;
			handHistory = gameDetails.handHistory ?? [];
			step = 4;
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
				position: position,
				playerCount: playerCount,
				card1: currentHand.card1,
				card2: currentHand.card2,
				suited: currentHand.suited,
				toCall: currentHand.toCall,
				action: currentHand.action,
				raiseAmount: currentHand.raiseAmount,
				results: currentHand.result,
				notes: currentHand.notes	
			});
			const key = `${player} - ${venue} - ${date}`;
			const value = JSON.stringify({ player, venue, date, handHistory });
			localStorage.setItem(key, value);
			currentHand = { suited: 'Offsuit', card1: null, card2: null, notes: ''};
		}
	
	// JJJJJJJJ   UU    UU MM      MM PPPPPPP             PPPPPPP      AA      GGGGGG   EEEEEEEE
	//    JJ      UU    UU MMM    MMM PP    PP            PP    PP   AA  AA   GG        EE
	//    JJ      UU    UU MM MMMM MM PPPPPPPP            PPPPPPPP  AAAAAAAA  GG   GGG  EEEEEE
	// J  JJ      UU    UU MM  MM  MM PP                  PP        AA    AA  GG     G  EE
	//  JJJ        UUUUUU  MM  MM  MM PP                  PP        AA    AA   GGGGGG   EEEEEEEE
	
		function goToPositionAndPlayer() {
			step = 2;
		}
	
		function goToCardSelection() {
			if (!currentHand.position || !currentHand.playerCount) {
				step = 2;
			}
			else {
			step = 3;}
	
		}
	
		function goToAction() {
			if (!currentHand.card2) {
				step = 3;	
			}
			else {
			step = 4;}
		}
	
		function goToResult() {
			step = 5;
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
			<div class="player-info">
				{#each gameHistory as game}
					<div class="side-by-side">
						<button class="card-info"  on:click={() => loadGame(game)}>
							{game}
						</button>
						<div class="delete-button">
							<button on:click={() => deleteGame(game)}>
								Delete
							</button>
						</div>
					</div>
				{/each}
				<div class="confirm">
					<button class="confirm-button" on:click={() => (step = 1)}>Add New Game</button>
				</div>
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
				<input id="player" type="text" bind:value={player} />
				<label for="venue">Venue:</label>
				<input id="venue" type="text" bind:value={venue} />
				<label for="date">Date:</label>
				<input id="date" type="date" bind:value={date} />
				<div class="confirm">
					<button class="confirm-button" on:click={confirmGameInfo}>Confirm</button>
				</div>
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
			<div class="Blinds">
				<div class="side-by-side">
					<label for="Blinds">Blinds:</label>
					<input id="SB" type="text" bind:value={currentHand.SB} />
					<span> / </span>
					<input id="BB" type="text" bind:value={currentHand.BB} />
				</div>
			</div>
			<div class="player-postion">
				<div class="player-selection">
					<h2>Players</h2>
					<ul>
						{#each players as p}
							<li>
								<button class={p === currentHand.playerCount ? 'active' : ''} on:click={() => handlePlayers(p)}>
									{p}
								</button>
							</li>
						{/each}
					</ul>
				</div>
				<div class="position-selection">
					<h2>Position</h2>
					<ul>
						{#each positions as pos}
							<li>
								<button class={pos === currentHand.position ? 'active' : ''} on:click={() => handlePosition(pos)}>
									{pos}
								</button>
							</li>
						{/each}
					</ul>
				</div>
			</div>
			<div class="confirm">
				<button class="confirm-button" on:click={confirmPlayerPosition}>Confirm</button>
			</div>
	
	<!--  CCCCCCC      AA     RRRRRRR   DDDDDD     SSSSSSS -->
	<!-- CC          AA  AA   RR    RR  DD    DD  SS       -->
	<!-- CC         AAAAAAAA  RRRRRRR   DD    DD   SSSSSS  -->
	<!-- CC         AA    AA  RR   RR   DD    DD        SS -->
	<!--  CCCCCCC   AA    AA  RR    RR  DDDDDD    SSSSSSS  -->
		{:else if step === 3}
			<div>
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
						<div class="backspace-button">
							<button on:click={() => handleUndoLastCard()}>
								<span class="backspace-icon">&#9003;</span>
							</button>
						</div>
						
	
					</div>
				</div>
				<div class="card-grid">
					{#each cards as card}
						<button on:click={() => handleCardSelection(card)}>{card}</button>
					{/each}
				</div>
				<div class="result-grid">
					{#each suited as s}
						<button on:click={() => handleToggleSuited(s)}>{s}</button>
					{/each}
				</div>
			</div>
	
	<!--    AA       CCCCCCC  TTTTTTTT  IIIIIIII   OOOOOO   NN    NN -->
	<!--  AA  AA    CC           TT        II     OO    OO  NNN   NN -->
	<!-- AAAAAAAA   CC           TT        II     OO    OO  NN NN NN -->
	<!-- AA    AA   CC           TT        II     OO    OO  NN  NNNN -->
	<!-- AA    AA    CCCCCCC     TT     IIIIIIII   OOOOOO   NN    NN -->
		{:else if step === 4}
			<div>
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
				<div class="confirm">
					<button class="confirm-button" on:click={confirmAction}>Confirm</button>
				</div>
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
	
			<div class="result-grid">
				{#each results as result}
					<button class="confirm-button" on:click={() => handleResult(result)}>{result}</button>
				{/each}
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
			<button on:click={goToAction} >{currentHand.toCall} to call, {currentHand.action} 
				{currentHand.action === 'Raise' ? currentHand.raiseAmount : ' '}</button>
			<button on:click={goToResult} >{currentHand.result}</button>
		</div>
		<div class="notes">
			<textarea rows="5" cols="50" id="notes" bind:value={currentHand.notes}></textarea>
		</div>
		<div class="confirm">
			<button class="confirm-button" on:click={addHand(currentHand.notes)}>Add Another Hand</button>
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
			max-width: 900px;
			margin: auto;
			padding: 20px;
			background: #d3d3d3;
			border-radius: 8px;
			box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
			display: flex;
			flex-direction: column;
			margin-bottom: 15px;
		}
	
		.container button {
			padding: 10px 20px;
			font-size: 16px;
			border: 1px solid #ccc;
			border-radius: 4px;
			background-color: #f8f9fa;
			color: #007bff;
			cursor: pointer;
			padding: 20px;
			margin-right: 5px;
			margin-left: 5px;
			margin-top: 5px;
			margin-bottom: 5px;
			width: 100%;
		}
	
		.container button:hover {
			background-color: #007bff;
			color: #f8f9fa;
		}
	
		.container label {
			font-weight: bold;
			margin-bottom: 5px;
		}
	
		.container input {
			padding: 10px;
			font-size: 16px;
			border: 1px solid #ccc;
			border-radius: 4px;
			box-sizing: border-box;
			width: 100%;
		}
	
		.container ul,
		li {
			list-style: none;
			padding: 0;
			margin: 0;
		}
	
		.confirm button {
			background-color: #28a745;
			color: #000000;
		}
	
		.side-by-side {
			display: flex;
			justify-content: space-between;
			width: 100%;
		}
	
		.side-by-side-grid {
			display: grid;
			grid-template-columns: repeat(2, auto);
			justify-content: space-between;
			width: 100%;
		}
	
	
		.delete-button button {
			background-color: #dc3545;
			color: #000000;
		}
	
		.gotoPage-button button{
			background-color: #829807;
			color: #000000;
			font-size: 24px;
		}
	
		.entry-display {
			font-size: 24px;
			border: 1px solid #ccc;
			border-radius: 4px;
			background-color: #ce9e52;
			margin-bottom: 20px;
			cursor: pointer;
			text-align: center;
			padding: 20px;
			margin-right: 10px;
			margin-left: 10px;
			margin-top: 10px;
			margin-bottom: 10px;
		}
	
		/* PPPPPPP     OOOOOO    SSSSSSS  IIIIIIII  TTTTTTTT  IIIIIIII   OOOOOO   NN    NN */
		/* PP    PP   OO    OO  SS           II        TT        II     OO    OO  NNN   NN */
		/* PPPPPPPP   OO    OO   SSSSSS      II        TT        II     OO    OO  NN NN NN */
		/* PP         OO    OO        SS     II        TT        II     OO    OO  NN  NNNN */
		/* PP          OOOOOO   SSSSSSS   IIIIIIII     TT     IIIIIIII   OOOOOO   NN    NN */
		.player-postion {
			display: grid;
			grid-template-columns: 1fr 1fr;
			gap: 20px;
		}
	
		/*  CCCCCCC      AA     RRRRRRR   DDDDDD     SSSSSSS */
		/* CC          AA  AA   RR    RR  DD    DD  SS       */
		/* CC         AAAAAAAA  RRRRRRR   DD    DD   SSSSSS  */
		/* CC         AA    AA  RR   RR   DD    DD        SS */
		/*  CCCCCCC   AA    AA  RR    RR  DDDDDD    SSSSSSS  */
		.card-grid {
			display: grid;
			grid-template-columns: repeat(4, auto);
			gap: 10px;
		}
	
		/* .material-icons {
			padding: 1px;
		} */
	
		.result-grid {
			display: grid;
			grid-template-columns: repeat(2, auto);
			gap: 10px;
		}
	
		textarea {
			padding: 10px;
			font-size: 24px;
			border: 1px solid #ccc;
			border-radius: 4px;
			background-color: #f8f9fa;
			margin-bottom: 20px;
			cursor: pointer;
			width: 100%;
			text-align: center;
	
		}
	
	.backspace-button button{
		background-color: #ce9e52;
		color: #000000;
		border-color: #ce9e52;
		height: 18px;
		width: 18px;
		transform: translateY(-60%);
	}
	
	.backspace-icon {
		background-color: #ce9e52;
		color: #c60d0d;
		font-size: 24px;
	}
	</style>