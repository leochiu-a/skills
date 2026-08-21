# skills

我自己在用的 [Claude Code](https://claude.com/claude-code) skills，圍繞著同一件事：**在寫任何 code 之前先把驗收標準（AC）釘死，然後用 TDD 一片一片做完，最後獨立審一遍 diff 再對著 AC 回報**。

## Skills

| Skill | 做什麼 | 什麼時候會觸發 |
| --- | --- | --- |
| [`implement`](skills/implement/SKILL.md) | 薄薄的 orchestrator：確認 AC → 交給 TDD → 審 diff → 對著 AC 回報 | 要求實作功能、修 bug，且不是一行就能改完的 |
| [`grill-me-ac`](skills/grill-me-ac/SKILL.md) | 用最少的來回問題把 AC 問清楚，拆成 machine-verifiable 和 human-judgment 兩類 | AC 還沒講明確的時候，可以單獨用，也會被 `/implement` 呼叫 |
| [`tdd`](skills/tdd/SKILL.md) | red-green-refactor，一次一個 vertical slice，測在 public seam 上而不是內部實作 | 說「用 TDD 實作」，或 `/implement` 確認完 AC 後交棒過來 |
| [`code-review`](skills/code-review/SKILL.md) | 三個獨立軸審 diff：AC 忠實度、repo 慣例、綠燈可不可信。三個 sub-agent 平行跑，結果不合併 | 說要 review 某個分支/PR/未提交的改動，或 `/tdd` 跑完之後 |

四個可以獨立用，也可以串起來——`/implement` 就是把另外三個串起來的那條線。

## 設計上的幾個取捨

- **AC 分兩桶**。machine-verifiable（測試、typecheck、build）可以自動驗；human-judgment（UX、文案、商業邏輯、風險）只能標記出來請人看。分開之後就不會出現「全部綠燈所以做完了」這種誤導性的回報。
- **不要空手問 AC**。先自己草擬一份，只針對真的判斷不了的地方發問，而且一次問完，不要一回合一個問題。
- **Vertical slicing**。一個 AC 項目走完 red → green 才換下一個，而不是把測試全部寫完再一起實作。Refactor 是全綠之後獨立的一輪。
- **測 seam，不測內部**。不 mock 內部協作者、不測 private method，這樣測試才活得過之後的重構。
- **綠燈要獨立驗**。`/tdd` 說「綠了」是它對自己工作的宣稱，而 exit code、CI、測試報告全都在測試的下游——測試本身被改動時，這些訊號會互相確認、然後一起錯。所以 `code-review` 有一軸專門只問：這個綠是實作換來的，還是測試被折出來的。
- **審的人不能是寫的人**。剛寫完 code 的那個 context 知道每個妥協當下為什麼看起來合理，它不能審自己的 diff，也不能負責替審查者定框架——所以三個軸都跑在獨立的 sub-agent，而且只拿到 fixed point 和 AC，其餘全部從 git 重新導出。實作者對自己做了什麼的說法，一律不採信。
- **三軸不合併、不重排**。一個改動可以慣例全對但做錯東西、可以完全照 AC 但破壞慣例、也可以又乾淨又照 AC 但綠燈是斷言放寬換來的。混成一個結論，就是讓其中一軸幫另一軸遮掩。

## 出處

`code-review` 的 two-axis 結構和 Fowler code smell baseline 改寫自 [mattpocock/skills](https://github.com/mattpocock/skills)（MIT，見 [NOTICE](NOTICE)）。AC 軸和 Green trust 軸是這個 repo 自己的。

## 安裝

用 [`skills` CLI](https://github.com/vercel-labs/skills) 裝，會互動式讓你選要裝哪幾個、裝到哪個 agent：

```bash
npx skills add leochiu-a/skills
```

裝到 user 層級（`~/.claude/skills/`）而不是只在當前專案：

```bash
npx skills add leochiu-a/skills --global
```

只裝其中一個，跳過所有確認：

```bash
npx skills add leochiu-a/skills --skill tdd -y
```

先看看這個 repo 裡有哪些 skill：

```bash
npx skills add leochiu-a/skills --list
```

不想用 CLI 的話，直接把資料夾複製過去也可以：

```bash
git clone https://github.com/leochiu-a/skills.git && cp -r skills/skills/* ~/.claude/skills/
```

裝好後在 Claude Code 裡用 `/implement`、`/grill-me-ac`、`/tdd` 呼叫，或是直接描述需求讓它自己觸發。
