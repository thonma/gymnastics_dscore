<script>
  // =========================================
	// 定数
	// =========================================
	const numFmt = new Intl.NumberFormat('ja-JP', { minimumFractionDigits: 1 });
	const difficulties = [...'ABCDEFGHIJ'];
	const groups = [...'1234'];
	// 種目は英語で event だが分かりにくくなるのでローマ字にする
	const shumokuList = ['ゆか', 'あん馬', 'つり輪', '跳馬', '平行棒', '鉄棒'];

	// =========================================
	// 選手情報
	// =========================================
	let player = {
		num: '',
		team: '',
		grade: '',
		fullname: '',
		note: '',
	};

	// =========================================
	// 演技構成
	// =========================================
	let engiKoseiList = shumokuList.map(s => {
		return {
			shumoku: s,
			engiKosei: new Array(s === '跳馬' ? 1 : 10).fill(0).map(() => {
				return {
					kumiawaseWithPrev: false,
					num: '',
					name: '',
					group: '',
					nando: '',
					// kumiawaseWithPrev: true,
					// num: '1',
					// name: '片腕支持3/4ひねり単棒倒立経過、軸手を換えて片腕支持3/4ひねり支持',
					// group: '2',
					// nando: 'C',
				};
			}),
			score: {
				nando: 0.0,
				group: 0.0,
				kumiawase: 0.0,
				d: 0.0, // nando + group + kumiawase
			},
		};
	});

	// =========================================
	// 演技構成のバリデーションエラーを探す
	// =========================================
	function findEngiKoseiValidationErrors() {
		const errs = [];
		// TODO 実装
		return errs;
	}

	// =========================================
	// Dスコアの算出
	// =========================================
	function computeDScore() {
		console.log(skills);

		difficultyAddScore = 0.0;
		groupAddScore = 0.0;
		combinationAddScore = 0.0;

		if (0 < findSkillsValidationErrors().length) {
			// TODO: エラー表示
			return;
		}

		// ========================================
		// 難度加点の算出
		// A=0.1, B=0.2 ... J=1.0 と変換して合計すると丸め誤差が出るため、
		// A=1, B=2 ... J=10 と変換して合計して、最後に10で割る。
		// ========================================
		difficultyAddScore = skills.reduce((p, skill) => {
			return p + difficulties.indexOf(skill.difficulty) + 1;
		}, 0);
		difficultyAddScore /= 10;

		// ========================================
		// 特別要求加点の算出
		// ========================================
		const selectedGroups = skills.map(s => s.group);
		groupAddScore += selectedGroups.includes('1') ? 0.5 : 0;
		groupAddScore += selectedGroups.includes('2') ? 0.5 : 0;
		groupAddScore += selectedGroups.includes('3') ? 0.5 : 0;
		if (0 < skills.filter(s => s.group === '4').length) {
			const finalSkill = skills.filter(s => s.group === '4')[0];
			if (finalSkill.difficulty === 'C') {
				groupAddScore += 0.3;
			} else {
				groupAddScore += 0.5;
			}
		}

		// ========================================
		// 組合せ加点の算出 (ゆか/鉄棒のみ)
		// 丸め誤差が出るため、整数で計算して最後に10で割る。
		// ========================================
		if (shumoku === 'ゆか') {
			// TODO
		} else if (shumoku === '鉄棒') {
			for (let i = 0; i < skills.length - 1; i++) {
				combinationAddScore += 10 * computeCombinationAddScore(shumoku, skills[i], skills[i + 1]);
			}
		}
		combinationAddScore /= 10;
	}

	// =========================================
	// 技ユーティリティ
	// =========================================
	function computeCombinationAddScore(shumoku, skill1, skill2) {
		if (skill2.combineWithPrev === false) {
			return 0.0;
		}

		if (shumoku === 'ゆか') {
			// TODO
			return 0.0;
		}

		if (shumoku === '鉄棒') {
			// 「C以上の手放し技 + C以上の手放し技」の組合せ
			if (skill1.group === '2' && skill2.group === '2' && 'C' <= skill1.difficulty && 'C' <= skill2.difficulty) {
				if (skill1.difficulty === 'C' || skill2.difficulty === 'C') {
					return 0.1;
				} else {
					return 0.2;
				}
			}
			// 「D以上のアドラー系 + D以上の手放し技」の組合せ
			if ('D' <= skill1.difficulty && isAdler(skill1.name) && skill2.group === '2' && 'D' <= skill2.difficulty) {
				if (skill2.difficulty === 'D') {
					return 0.1;
				} else {
					return 0.2;
				}
			}
		}

		return 0.0;
	}

	function isNotTwist(skillName) {
		const words = ['ひねり', 'ハーフ'];
		return words.some(w => skillName.includes(w)) === false;
	}

	function isSingle(skillName) {
		return isDoubleOrMore(skillName) === false;
	}

	function isDoubleOrMore(skillName) {
		const words = ['ダブル', '二回宙', '2回宙', '２回宙', '三回宙', '3回宙', '３回宙',];
		return words.some(w => skillName.includes(w));
	}

	function isAdler(skillName) {
		const words = ['アドラー',];
		return words.some(w => skillName.includes(w));
	}
</script>

<svelte:head>
	<title>Home</title>
</svelte:head>

<section>
	<details class="bg-slate-100 p-2">
		<summary>選手情報</summary>

		<div class="text-sm text-red-500 mb-2">
			<!-- TODO -->
			エラーメッセージ01<br>
			エラーメッセージ02<br>
			エラーメッセージ03<br>
		</div>

		<div class="flex justify-between">
			<div class="w-[10%]">
				<label for="playerNum" class="block text-sm">選手番号</label>
				<input type="text" name="playerNum" bind:value={player.num} class="w-full px-1 border" />
			</div>

			<div class="w-[30%]">
				<label for="playerTeam" class="block text-sm">所属</label>
				<input type="text" name="playerTeam" bind:value={player.team} class="w-full px-1 border" />
			</div>

			<div class="w-[10%]">
				<label for="playerGrade" class="block text-sm">学年</label>
				<input type="text" name="playerGrade" bind:value={player.grade} class="w-full px-1 border" />
			</div>

			<div class="w-[49%]">
				<label for="playerFullname" class="block text-sm">氏名</label>
				<input type="text" name="playerFullname" bind:value={player.fullname} class="w-full px-1 border" />
			</div>
		</div>

		<label for="playerNote" class="block text-sm">備考</label>
		<textarea name="playerNote" rows="2" bind:value={player.note} class="w-full px-1 border" />
	</details>
	<hr class="my-2">

	<div>
		<div class="text-sm text-red-500 mb-2">
			<!-- TODO -->
			エラーメッセージ01<br>
			エラーメッセージ02<br>
			エラーメッセージ03<br>
		</div>

		<div class="flex justify-between items-start">
			{#each engiKoseiList as ek, ekIndex}
			<table class="mx-2 w-full" class:ml-0={ekIndex === 0} class:mr-0={ekIndex === engiKoseiList.length - 1}>
				<thead>
					<tr>
						<th class="text-left " colspan="2">{ ek.shumoku }</th>
					</tr>
				</thead>
				<tbody>
					{#each engiKoseiList[ekIndex].engiKosei as engiKosei, engiKoseiIndex}
					<tr>
						<td class="border rounded p-2" colspan="2">
							{#if engiKosei.num !== ''}
								{ `${engiKosei.name} (${engiKosei.group}${engiKosei.nando})` }
							{:else}
								<button class="bg-slate-100 border rounded px-2 py-1 w-full">技を選択する</button>
							{/if}
						</td>
					</tr>
					{#if ek.shumoku !== '跳馬' && engiKoseiIndex < 9}
					<tr>
						<!-- "↓"をアイコンにする -->
						<td class="font-bold text-lg text-center text-slate-300" colspan="2">↓</td>
					</tr>
					{/if}
					{/each}
					{#if ek.shumoku !== '跳馬'}
					<tr>
						<td class="pt-2 pl-2 font-bold text-lg">難度点</td>
						<td class="pt-2 pr-2 font-bold text-lg text-right">{ ek.score.nando.toFixed(1) }</td>
					</tr>
					<tr>
						<td class="pl-2 font-bold text-lg">グループ要求加点</td>
						<td class="pr-2 font-bold text-lg text-right">{ ek.score.nando.toFixed(1) }</td>
					</tr>
					<tr class="text-transparent" class:text-current={ ek.shumoku === 'ゆか' || ek.shumoku === '鉄棒' }>
						<td class="pl-2 font-bold text-lg">組合せ加点</td>
						<td class="pr-2 font-bold text-lg text-right">{ ek.score.kumiawase.toFixed(1) }</td>
					</tr>
					{/if}
					<tr>
						<td class="pt-2 pl-2 font-bold text-lg">Dスコア</td>
						<td class="pt-2 pr-2 font-bold text-lg text-right">{ ek.score.d.toFixed(1) }</td>
					</tr>
					{#if ek.shumoku !== '跳馬'}
					<tr>
						<td class="py-2 pl-2 font-bold text-lg">技数減点</td>
						<td class="py-2 pr-2 font-bold text-lg text-right">0.0</td>
					</tr>
					{/if}
				</tbody>
			</table>
			{/each}
		</div>
	</div>

	<hr class="my-2">
	<details class="bg-slate-100 p-2">
			<summary>Dump</summary>

			<pre><code>player = { JSON.stringify(player, null, 2) }

engiKoseiList = { JSON.stringify(engiKoseiList, null, 2) }
</code></pre>
	</details>
</section>

<style>
</style>
