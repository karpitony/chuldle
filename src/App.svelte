<svelte:head>
  <title>Chuldle</title>
</svelte:head>

<style global lang="postcss">
	@tailwind base;
	@tailwind components;
	@tailwind utilities;

    .backdrop {
    width: 100%;
    height: 100%;
    position: fixed;
    top: 0;
    left: 0;
    background: rgba(0, 0, 0, 0.8);
    }
    .modal {
    padding: 30px;
    border-radius: 10px;
    width: 400px;
    margin: 10% auto;
    text-align: center;
    background: white;
    }
    .succ .modal {
    background: green;
    color: white;
    }
    .fail .modal {
    background: crimson;
    color: white;
    }

	.title {
		font-family: 'Comic Sans MS', cursive;
		font-size: 2em;
	}
	.graybutton {
		background: gray;
		color: white;
		padding: 5px 10px;
	    border-radius: 10px;
	}

	.github-link img {
		width: 40px; /* Adjust the size as needed */
		height: auto;
		margin-right: 5px;
		vertical-align: middle;
	}
	/* Style for the How to Play button */
	.how-to-play-button {
		background-color: #4CAF50; /* Green background */
		color: white; /* White text */
		padding: 10px 20px; /* Top/bottom padding of 10px, left/right padding of 20px */
		border: none; /* No border */
		text-align: center; /* Center text */
		text-decoration: none; /* No underline */
		display: inline-block; /* Allow inline block level properties */
		font-size: 16px; /* Set font size */
		margin: 2px 2px; /* Margin around the button */
		cursor: pointer; /* Pointer cursor on hover */
		border-radius: 8px; /* Rounded corners */
		transition-duration: 0.4s; /* Smooth transition for hover effect */
	}

	/* Hover effect for the How to Play button */
	.how-to-play-button:hover {
		background-color: white; /* White background */
		color: #4CAF50; /* Green text */
		border: 1px solid #4CAF50; /* Green border */
	}

</style>

<script>
	import * as data from './data.json'
	import Cell from './Cell.svelte';


	const process = (word) => {
		return word.split('').reduce( (prv, cur) => [...prv, getConstantVowel(cur)], []).flat();
	};

	// Simple seedable LCG (Linear Congruential Generator)
	function LCG(seed) {
		return function() {
			seed = (seed * 1879 + 1013904223) % 65536;
			return seed / 65536;
		};
	}
	function fisherYatesShuffle(array, randomFunc) {
		let i, j, temp;
		for (i = array.length - 1; i > 0; i--) {
			j = Math.floor(randomFunc() * (i + 1));
			temp = array[i];
			array[i] = array[j];
			array[j] = temp;
		}
		return array;
	}


	const stations = data["DATA"];
	const easyStations = stations.filter((element) => process(element["station_nm"]).length <= 15);
	const names = stations.map((element) => element["station_nm"]);

	// Use the current day as the seed
	const epochMs = new Date('January 1, 2022 00:00:00').valueOf();
	const now = Date.now();
	const msInDay = 86400000;
	const daysSinceEpoch = Math.floor((now - epochMs) / msInDay);
	const seed = daysSinceEpoch;
	const random = LCG(seed);


	const shuffledEasyStations = fisherYatesShuffle([...easyStations], random);
	let practiceMode = false;
	const randomizeSolution = () => {
		const randomIndex = Math.floor(Math.random() * easyStations.length);
		return easyStations[randomIndex]["station_nm"];
	};

	var solution = shuffledEasyStations[daysSinceEpoch % shuffledEasyStations.length]["station_nm"];
    function togglePracticeMode() {
        practiceMode = !practiceMode;
        // Optionally reinitialize the game state here if needed
		// Reinitialize game state
        currentGuess = '';
        guesses = [];
        remain = new Array(life).fill(0);
        currentStatus = 0;

        // Update solution based on practice mode
        if (practiceMode) {
            solution = randomizeSolution();
        } else {
            solution = shuffledEasyStations[daysSinceEpoch % shuffledEasyStations.length]["station_nm"];
        }

        // Also update processed solution display
        disSolution = process(solution);
    }

	const life = 6;
	let remain = new Array(life).fill(0);

	let currentStatus = 0;

	var disSolution = process(solution);
	const emptyInfo = {letter: '',matched: 0};
	let currentGuess = '';
	let guesses = [];
	function stackGuesses() {
		if(!names.includes(currentGuess)){
			alert('역 이름이 잘못되었습니다.');
			return;
		}
		if(process(currentGuess).length != disSolution.length){
			alert('역의 자모 수가 일치하지 않습니다.');
			return;
		}
		let disassembled = calc(process(currentGuess), disSolution);
		
		guesses = [...guesses, {guess: currentGuess, disassembled: disassembled}];

		if(disassembled.filter( (element) => element.matched !== 1).length === 0){
			currentStatus=1;
		}
		else if(guesses.length == life){
			currentStatus=2;
		}
		console.log(currentStatus);
		currentGuess = '';
		remain = remain.slice(1);
	}
	const onKeyPress = e => {
		if (e.charCode === 13) stackGuesses();
	};
	const calc = (arr, dis) => {
		let len = arr.length;
		let res = new Array(arr.length).fill(0);
		let vis = new Array(arr.length).fill(0);

		let ans = []
		for (let i = 0; i < len; i++) {
			if(arr[i] === dis[i]){
				res[i] = 1;
				vis[i] = 1;
			}
		}
		for (let i = 0; i < len; i++) {
			for (let j = 0; j < len; j++) {
				if(arr[i] === dis[j] && res[i] === 0 && vis[j] === 0){
					res[i] = 2;
					vis[j] = 1;
				}
			}
			ans.push({letter: arr[i], matched: res[i]});
		}
		return ans;
	};
	let showIntro = false;
	function toggleIntro(){
		showIntro = !showIntro;
	}
	function getConstantVowel(kor) {
		if (kor < '가' || kor > '힣'){
			return [kor];
		}
		const f = ['ㄱ', 'ㄲ', 'ㄴ', 'ㄷ', 'ㄸ', 'ㄹ', 'ㅁ',
				'ㅂ', 'ㅃ', 'ㅅ', 'ㅆ', 'ㅇ', 'ㅈ', 'ㅉ',
				'ㅊ', 'ㅋ', 'ㅌ', 'ㅍ', 'ㅎ'];
		const s = ['ㅏ', 'ㅐ', 'ㅑ', 'ㅒ', 'ㅓ', 'ㅔ', 'ㅕ',
				'ㅖ', 'ㅗ', 'ㅘ', 'ㅙ', 'ㅚ', 'ㅛ', 'ㅜ',
				'ㅝ', 'ㅞ', 'ㅟ', 'ㅠ', 'ㅡ', 'ㅢ', 'ㅣ'];
		const t = ['', 'ㄱ', 'ㄲ', 'ㄳ', 'ㄴ', 'ㄵ', 'ㄶ',
				'ㄷ', 'ㄹ', 'ㄺ', 'ㄻ', 'ㄼ', 'ㄽ', 'ㄾ',
				'ㄿ', 'ㅀ', 'ㅁ', 'ㅂ', 'ㅄ', 'ㅅ', 'ㅆ',
				'ㅇ', 'ㅈ', 'ㅊ', 'ㅋ', 'ㅌ', 'ㅍ', 'ㅎ'];

		const ga = 44032;
		let uni = kor.charCodeAt(0);

		uni = uni - ga;

		let fn = parseInt(uni / 588);
		let sn = parseInt((uni - (fn * 588)) / 28);
		let tn = parseInt(uni % 28);

		if(t[tn]!='')return [f[fn],s[sn],t[tn]];
		return [f[fn],s[sn]]
	}

	const GREEN_BOX = '🟩';
	const YELLOW_BOX = '🟨';
	const GRAY_BOX = '⬜';

	function generateSummary() {
		let summary = `Chuldle 결과\nhttps://chuldle.netlify.app/`;
		if(practiceMode){
			summary += `\nPractice\nAnswer: ${solution}\n`;
		}
		else{
			const gameNumber = seed - 723;
			summary += `\nChuldle #${gameNumber}\n`;
		}
		guesses.forEach(guess => {
			guess.disassembled.forEach(letterInfo => {
				switch (letterInfo.matched) {
					case 1:
						summary += GREEN_BOX;
						break;
					case 2:
						summary += YELLOW_BOX;
						break;
					default:
						summary += GRAY_BOX;
				}
			});
			summary += '\n'; // New line for each guess
		});
		return summary.trim(); // Remove the last new line
	}

	function shareResult() {
		const summary = generateSummary();
		navigator.clipboard.writeText(summary).then(() => {
			alert('Result copied to clipboard!');
		}, (err) => {
			alert('Failed to copy result: ', err);
		});
	}

	function newPractice() {
		togglePracticeMode();
		togglePracticeMode();
	}
</script>


{#if practiceMode==0}
	<div class="title flex justify-center">
		Chuldle
	</div>
{/if}
{#if practiceMode==1}
	<div class="title flex justify-center">
		Chuldle practice
	</div>
{/if}

<div>
    {#each guesses as {guess, disassembled}}
		<div class="flex justify-center mb-1">
			{#each disassembled as letter}
				<Cell info={letter}/>
			{/each}
		</div>
    {/each}

    {#each remain as {_}}
		<div class="flex justify-center mb-1">
			{#each disSolution as letter}
				<Cell info={emptyInfo}/>
			{/each}
		</div>
    {/each}

</div>



{#if currentStatus === 1 || currentStatus === 2}
	<div class="backdrop" class:succ={currentStatus===1} class:fail={currentStatus===2}>
		<div class="modal">
			<p>{currentStatus===1 ? "Success" : "Failure"}</p>
			<p>{currentStatus===1 ? guesses.length + " guesses" : "Answer: " + solution}</p>
			<button on:click={shareResult} class="graybutton">Share</button>

			{#if practiceMode==1}
				<button on:click={newPractice} class="graybutton">New Game</button>
			{/if}
		</div>
	</div>
{/if}

{#if showIntro}
    <div class="backdrop">
		<div class="modal">
			<p>Chuldle은 서울 지하철 역을 맞추는 게임입니다.</p>
			<p>답이 동작일 때, 행당이라고 입력한 경우 아래와 같이 표시됩니다.</p>
			<div class="flex justify-center mb-1">
				{#each calc(process('행당'), process('동작')) as letter}
					<Cell info={letter}/>
				{/each}
			</div>
			<p>답이 종로3가일 때, 종로4가라고 입력한 경우 아래와 같이 표시됩니다.</p>
			<div class="flex justify-center mb-1">
				{#each calc(process('종로3가'), process('종로4가')) as letter}
					<Cell info={letter}/>
				{/each}
			</div>
			<p>입력한 역 이름의 자모 수가 네모칸의 개수와 다르면 오류 메시지가 뜹니다.</p>
			<p>서울 지하철역 이름이 아닌 값을 입력해도 오류 메시지가 뜨게 됩니다.</p>
			<p></p>
			<p>정답은 <b>매일 자정</b> 업데이트되며, 하루 동안은 바뀌지 않습니다.</p>
			<button on:click={toggleIntro} class="graybutton">close</button>
		</div>
    </div>
{/if}


<div class="flex justify-center mt-5">
    <input 
        on:keypress={onKeyPress}
        bind:value={currentGuess}
    >
    <button on:click={stackGuesses}>
        Enter
    </button>
</div>

<div class="flex justify-center mt-5">
    <!-- Update the text within the button -->
    <button class="how-to-play-button" on:click={toggleIntro}>
        How to Play
    </button>
    <!-- GitHub link remains the same -->
    <a href="https://github.com/ainta/chuldle" target="_blank" class="github-link">
        <img src="github-mark.png" alt="GitHub" />
    </a>
</div>
