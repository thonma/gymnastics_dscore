<script lang="ts">
	class Waza {
		shumoku: 'ゆか' | 'あん馬' | 'つり輪' | '跳馬' | '平行棒' | '鉄棒';
		num: number;
		formalName: string; // 正式名称 (例: 後方かかえ込み2回宙返り1回ひねり)
		originatorName: string; // 人名 (例: ツカハラ)
		commonName: string[]; // 俗称 (例: 月面宙返り、ムーンサルト、ムーンサルト)
		group: 1|2|3|4;
		nando: `A`|`B`|`C`|`D`|`E`|`F`|`G`|`H`|`I`|`J`;

		constructor(
			shumoku: 'ゆか' | 'あん馬' | 'つり輪' | '跳馬' | '平行棒' | '鉄棒',
			num: number,
			formalName: string,
			originatorName: string,
			commonName: string[],
			group: 1|2|3|4,
			nando: `A`|`B`|`C`|`D`|`E`|`F`|`G`|`H`|`I`|`J`
		) {
			this.shumoku = shumoku;
			this.num = num;
			this.formalName = formalName;
			this.originatorName = originatorName;
			this.commonName = commonName;
			this.group = group;
			this.nando = nando;
		}
	}

	class EngiKoseiBase {
		wazaList: Waza[];

		constructor(wazaList: Waza[]) {
			this.wazaList = wazaList;
		}

		/**
		 * 入力チェック
		 * ・10技以下
		 * ・グループ1/2/3は4技以下
		 * ・グループ4は1技以下
		 * ・技番号の重複が無いこと
		 * 
		 * @returns OK=空配列、NG=エラーメッセージ配列
		 */
		validate(): boolean | string[] {
			// TODO 実装
			return [];
		}

		culculateNandoTen(): number {
			const kachiTenMap = { A: 1, B: 2, C: 3, D: 4, E: 5, F: 6, G: 7, H: 8, I: 9, J: 10 };
			const kachiTenSum = this.wazaList.reduce((prev, curr) => prev + (kachiTenMap[curr.nando] || 0), 0);
			return kachiTenSum / 10;
		}

		culculateGroupYokyuKaten(): number {
			let groupYokyuKaten = 0;

			const selectedGroupList = this.wazaList.map((waza) => waza.group);
			if (selectedGroupList.includes(1)) {
				groupYokyuKaten += 5;
			}
			if (selectedGroupList.includes(2)) {
				groupYokyuKaten += 5;
			}
			if (selectedGroupList.includes(3)) {
				groupYokyuKaten += 5;
			}

			const lastWaza = this.wazaList.find((waza) => waza.group === 4);
			if (lastWaza) {
				if (lastWaza.nando === 'A') {
					groupYokyuKaten += 1;
				} else if (lastWaza.nando === 'B') {
					groupYokyuKaten += 2;
				} else if (lastWaza.nando === 'C') {
					groupYokyuKaten += 3;
				} else {
					groupYokyuKaten += 5;
				}
			}

			return groupYokyuKaten / 10;
		}

		culculateKumiawaseKaten(): number {
			return 0.0;
		}

		culculateDScore(): number {
			const n = this.culculateNandoTen();
			const g = this.culculateGroupYokyuKaten();
			const k = this.culculateKumiawaseKaten();
			return n + g + k;
		}

		calculateGisuGenten(): number {
			if (8 <= this.wazaList.length) {
				return 0;
			}
			return 10 - this.wazaList.length;
		}
	}

	class EngiKoseiFX extends EngiKoseiBase {
		override validate(): boolean | string[] {
			// TODO 実装
			return [];
		}

		override culculateKumiawaseKaten() {
			// TODO 実装
			return 0.0;
		}
	}

	class EngiKoseiPH extends EngiKoseiBase {
	}

	class EngiKoseiSR extends EngiKoseiBase {
	}

	class EngiKoseiVT extends EngiKoseiBase {
		override validate(): boolean | string[] {
			// TODO 実装
			return [];
		}

		override culculateDScore(): number {
			// TODO 実装
			return 0.0;
		}
	}

	class EngiKoseiPB extends EngiKoseiBase {
	}

	class EngiKoseiHB extends EngiKoseiBase {
		override culculateKumiawaseKaten() {
			// TODO 実装
			return 0.0;
		}
	}

	class Player {
		num: string;
		team: string;
		grade: string;
		fullname: string;
		note: string;

		constructor(
			num: string,
			team: string,
			grade: string,
			fullname: string,
			note: string,
		) {
			this.num = num;
			this.team = team;
			this.grade = grade;
			this.fullname = fullname;
			this.note = note;
		}

		validate(): boolean | string[] {
			// TODO 実装
			return true;
		}
	}

	// =========================================
	// 定数
	// =========================================
	const nandoList: string[] = [...'ABCDEFGHIJ'];
	const groupList: string[] = [...'1234'];
	const shumokuList: string[] = ['ゆか', 'あん馬', 'つり輪', '跳馬', '平行棒', '鉄棒'];

	// =========================================
	// 選手情報
	// =========================================
	let player = {
		num: '',
		team: '',
		grade: '',
		fullname: '',
		note: ''
	};

	// =========================================
	// 演技構成
	// =========================================
	let engiKoseiList = shumokuList.map((s) => {
		return {
			shumoku: s,
			engiKosei: new Array(s === '跳馬' ? 1 : 10).fill(0).map(() => {
				return {
					kumiawaseWithPrev: false,
					num: '',
					name: '',
					group: '',
					nando: ''
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
				d: 0.0 // nando + group + kumiawase
			}
		};
	});

	// =========================================
	// 技選択
	// =========================================
	function createWazaDummy() {
		return [
			...[...new Array(40)].map((_, i) => ({
				shumoku: `ゆか`,
				num: i + 1,
				name: `ゆか_技_${i + 1}`,
				common_name: `俗称_${i + 1}`,
				group: groupList[i % 4],
				nando: nandoList[i % 10]
			})),
			...[...new Array(40)].map((_, i) => ({
				shumoku: `あん馬`,
				num: i + 1,
				name: `あん馬_技_${i + 1}`,
				common_name: `俗称_${i + 1}`,
				group: groupList[i % 4],
				nando: nandoList[i % 10]
			})),
			...[...new Array(40)].map((_, i) => ({
				shumoku: `つり輪`,
				num: i + 1,
				name: `つり輪_技_${i + 1}`,
				common_name: `俗称_${i + 1}`,
				group: groupList[i % 4],
				nando: nandoList[i % 10]
			})),
			...[...new Array(40)].map((_, i) => ({
				shumoku: `跳馬`,
				num: i + 1,
				name: `跳馬_技_${i + 1}`,
				common_name: `俗称_${i + 1}`,
				group: groupList[i % 4],
				nando: nandoList[i % 10]
			})),
			...[...new Array(40)].map((_, i) => ({
				shumoku: `平行棒`,
				num: i + 1,
				name: `平行棒_技_${i + 1}`,
				common_name: `俗称_${i + 1}`,
				group: groupList[i % 4],
				nando: nandoList[i % 10]
			})),
			...[...new Array(40)].map((_, i) => ({
				shumoku: `鉄棒`,
				num: i + 1,
				name: `鉄棒_技_${i + 1}`,
				common_name: `俗称_${i + 1}`,
				group: groupList[i % 4],
				nando: nandoList[i % 10]
			}))
		].sort((a, b) => {
			// ゆか、あん馬、つり輪、跳馬、平行棒、鉄棒の順
			if (a.shumoku !== b.shumoku) {
				return shumokuList.indexOf(a.shumoku) - shumokuList.indexOf(b.shumoku);
			}
			// 技グループ(1/2/3/4)の昇順
			if (a.group !== b.group) {
				return a.group - b.group;
			}
			// 技番号の昇順
			return a.num - b.num;
		});
	}
	let originalWazaList = createWazaDummy();
	console.log(originalWazaList);
	let searchResultWazaList = structuredClone(originalWazaList);
	let wazaSearchKeyword = '';
	function filterWazaList() {
		searchResultWazaList = searchResultWazaList.filter((w) => {
			return w.name.includes(wazaSearchKeyword);
		});
	}

	/**
	 * グループ要求加点の算出
	 *
	 * @param engiKosei
	 */
	function calculateGroupYokyuKaten(engiKosei) {
		let groupYokyuKaten = 0;

		const selectedGroupList = engiKosei.map((waza) => waza.group);
		if (selectedGroupList.includes('1')) {
			groupYokyuKaten += 5;
		}
		if (selectedGroupList.includes('2')) {
			groupYokyuKaten += 5;
		}
		if (selectedGroupList.includes('3')) {
			groupYokyuKaten += 5;
		}

		const lastWaza = engiKosei.find((waza) => waza.group === '4');
		if (lastWaza) {
			if (lastWaza.nando === 'A') {
				groupYokyuKaten += 1;
			} else if (lastWaza.nando === 'B') {
				groupYokyuKaten += 2;
			} else if (lastWaza.nando === 'C') {
				groupYokyuKaten += 3;
			} else {
				groupYokyuKaten += 5;
			}
		}

		return groupYokyuKaten / 10;
	}

	/**
	 * 組合せ加点の算出
	 *
	 * @param engiKosei
	 */
	function calculateKumiawaseKaten(engiKosei) {
		// TODO 実装
	}

	/**
	 * Dスコアの算出
	 *
	 * @param engiKosei
	 */
	function calculateDScore(engiKosei) {}

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
			if (
				skill1.group === '2' &&
				skill2.group === '2' &&
				'C' <= skill1.difficulty &&
				'C' <= skill2.difficulty
			) {
				if (skill1.difficulty === 'C' || skill2.difficulty === 'C') {
					return 0.1;
				} else {
					return 0.2;
				}
			}
			// 「D以上のアドラー系 + D以上の手放し技」の組合せ
			if (
				'D' <= skill1.difficulty &&
				isAdler(skill1.name) &&
				skill2.group === '2' &&
				'D' <= skill2.difficulty
			) {
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
		return words.some((w) => skillName.includes(w)) === false;
	}

	function isSingle(skillName) {
		return isDoubleOrMore(skillName) === false;
	}

	function isDoubleOrMore(skillName) {
		const words = ['ダブル', '二回宙', '2回宙', '２回宙', '三回宙', '3回宙', '３回宙'];
		return words.some((w) => skillName.includes(w));
	}

	function isAdler(skillName) {
		const words = ['アドラー'];
		return words.some((w) => skillName.includes(w));
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
			エラーメッセージ01<br />
			エラーメッセージ02<br />
			エラーメッセージ03<br />
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
				<input
					type="text"
					name="playerGrade"
					bind:value={player.grade}
					class="w-full px-1 border"
				/>
			</div>

			<div class="w-[49%]">
				<label for="playerFullname" class="block text-sm">氏名</label>
				<input
					type="text"
					name="playerFullname"
					bind:value={player.fullname}
					class="w-full px-1 border"
				/>
			</div>
		</div>

		<label for="playerNote" class="block text-sm">備考</label>
		<textarea name="playerNote" rows="2" bind:value={player.note} class="w-full px-1 border" />
	</details>
	<hr class="my-2" />

	<div>
		<div class="text-sm text-red-500 mb-2">
			<!-- TODO -->
			エラーメッセージ01<br />
			エラーメッセージ02<br />
			エラーメッセージ03<br />
		</div>

		<div class="flex justify-between items-start">
			{#each engiKoseiList as ek, ekIndex}
				<table
					class="mx-2 w-full"
					class:ml-0={ekIndex === 0}
					class:mr-0={ekIndex === engiKoseiList.length - 1}
				>
					<thead>
						<tr>
							<th class="text-left" colspan="2">{ek.shumoku}</th>
						</tr>
					</thead>
					<tbody>
						{#each engiKoseiList[ekIndex].engiKosei as engiKosei, engiKoseiIndex}
							<tr>
								<td class="border rounded p-2" colspan="2">
									{#if engiKosei.num !== ''}
										{`${engiKosei.name} (${engiKosei.group}${engiKosei.nando})`}
									{:else}
										<input
											type="text"
											name={`${ek.shumoku}_${engiKoseiIndex}`}
											list="WazaList"
											class="w-full px-2 py-1 border"
										/>
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
								<td class="pt-2 pr-2 font-bold text-lg text-right">{ek.score.nando.toFixed(1)}</td>
							</tr>
							<tr>
								<td class="pl-2 font-bold text-lg">グループ要求加点</td>
								<td class="pr-2 font-bold text-lg text-right">{ek.score.nando.toFixed(1)}</td>
							</tr>
							<tr
								class="text-transparent"
								class:text-current={ek.shumoku === 'ゆか' || ek.shumoku === '鉄棒'}
							>
								<td class="pl-2 font-bold text-lg">組合せ加点</td>
								<td class="pr-2 font-bold text-lg text-right">{ek.score.kumiawase.toFixed(1)}</td>
							</tr>
						{/if}
						<tr>
							<td class="pt-2 pl-2 font-bold text-lg">Dスコア</td>
							<td class="pt-2 pr-2 font-bold text-lg text-right">{ek.score.d.toFixed(1)}</td>
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

	<datalist id="WazaList">
		{#each searchResultWazaList as waza}
			<option value={waza.name + (waza.common_name ? ` (${waza.common_name})` : ``)}
				>{`グループ${waza.group} ${waza.nando}難度`}</option
			>
		{/each}
	</datalist>

	<hr class="my-2" />
	<details class="bg-slate-100 p-2">
		<summary>Dump</summary>

		<pre><code
				>player = {JSON.stringify(player, null, 2)}

engiKoseiList = {JSON.stringify(engiKoseiList, null, 2)}
</code></pre>
	</details>
</section>

<style>
</style>
