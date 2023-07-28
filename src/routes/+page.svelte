<script>
  // =========================================
	// 定数
	// =========================================
	const numFmt = new Intl.NumberFormat('ja-JP', { minimumFractionDigits: 1 });
	const difficulties = [...'ABCDEFGHIJ'];
	const groups = [...'1234'];
	// 種目は英語でeventだが分かりにくくなるのでローマ字にする
	const shumokuList = ['ゆか', 'あん馬', 'つり輪', '平行棒', '鉄棒'];

	// =========================================
	// メタデータ(所属、氏名など)
	// =========================================
	let metadata = 'ジムナス大学\n体操 太郎';

	// =========================================
	// 演技構成
	// =========================================
	let skills = new Array(10).fill().map(() => ({
		combineWithPrev: false,
		name: '',
		difficulty: '',
		group: '',
		note: '',
	}));

	// =========================================
	// 種目と各スコア
	// =========================================
	let shumoku = 'ゆか';
	let difficultyAddScore = 0.0;
	let groupAddScore = 0.0;
	let combinationAddScore = 0.0;
	let adjustmentScore = 0.0;

	// =========================================
	// 演技構成のバリデーションエラーを探す
	// =========================================
	function findSkillsValidationErrors() {
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
			if (finalSkill.difficulty === 'A') {
				groupAddScore += 0.1;
			} else if (finalSkill.difficulty === 'B') {
				groupAddScore += 0.2;
			} else if (finalSkill.difficulty === 'C') {
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
		const words = ['アドラー'];
		return words.some(w => skillName.includes(w));
	}
</script>

<svelte:head>
	<title>Home</title>
</svelte:head>

<section>
  <label for="metadata">所属、氏名など</label>
	<textarea bind:value={metadata} id="metadata"></textarea>

	<label for="shumoku">種目</label>
	<select bind:value={shumoku} on:change={computeDScore} id="shumoku">
		{#each shumokuList as s}
		<option value={s}>{s}</option>
		{/each}
	</select>

	<div>
	  <table>
			<thead>
				<tr>
					<th>No</th>
					{#if shumoku === 'ゆか' || shumoku === '鉄棒'}
					<th>組合せ</th>
					{/if}
					<th>技名</th>
					<th colspan="2">グループ/難度</th>
					<th>備考</th>
				</tr>
			</thead>
			<tbody>
				{#each skills as s, sIdx}
			  <tr>
				  <td>{sIdx + 1}</td>
					{#if shumoku === 'ゆか' || shumoku === '鉄棒'}
					<td>
						{#if 0 < sIdx}
					  <input type="checkbox" bind:checked={skills[sIdx].combineWithPrev} on:change={computeDScore} title="前の技と組み合わせて実施する場合はチェックを入れてください。">
						{/if}
					</td>
					{/if}
					<td>
					  <input type="text" bind:value={skills[sIdx].name}>
					</td>
					<td>
					  <select bind:value={skills[sIdx].group} on:change={computeDScore}>
							<option value=""></option>
							{#each groups as g}
							<option value={g}>{g}</option>
							{/each}
						</select>
					</td>
					<td>
					  <select bind:value={skills[sIdx].difficulty} on:change={computeDScore}>
							<option value=""></option>
							{#each difficulties as d}
							<option value={d}>{d}</option>
							{/each}
						</select>
					</td>
					<td>
					  <input type="text" bind:value={skills[sIdx].note}>
					</td>
				</tr>
				{/each}
			</tbody>
		</table>
	</div>

	<hr>

	<div>
	  <div>難度加点 <strong>{numFmt.format(difficultyAddScore)}</strong></div>
		<div>特別要求加点 <strong>{numFmt.format(groupAddScore)}</strong></div>
		{#if shumoku === 'ゆか' || shumoku === '鉄棒'}
		<div>組合せ加点 <strong>{numFmt.format(combinationAddScore)}</strong></div>
		{/if}
		<div>調整点 <input type="number" step="0.1" bind:value={adjustmentScore}></div>
		<div>Dスコア: <strong>{numFmt.format(difficultyAddScore + groupAddScore + combinationAddScore + adjustmentScore)}</strong></div>
	</div>

	<hr>

	<div>TODO: 免責事項(何があっても責任取りません的なやつ)/許諾事項(WTFPL的なやつ)/諸注意(対応していないルールなど)を書く</div>
	<div>TODO: GitHubへのアイコンリンクを実装</div>
</section>

<style>
  [id="metadata"] {
		width: 100%;
		height: 3rem;
		resize: vertical;
	}

	table {
		width: 100%;
	}

	table input {
		width: 100%;
	}

	table select {
		width: 100%;
	}
</style>
