# FinFlag: check a finance video before you act on it

**Track:** AI + Finance (my own idea)
**Author:** Seeun Park (individual project)

## The problem
Many teenagers get their first investing advice from TikTok and YouTube "finfluencers." A teen who hears "this stock will explode, buy before Monday" has no quick way to tell hype from honest analysis before putting real money in.

## What I built
FinFlag takes the transcript of a finance video and gives back:
- the **red flags** it finds: guaranteed returns, pressure to act now, a recommendation with no risks mentioned, predictions stated as facts, promotion of a paid product, and encouragement of risky bets such as options or short squeezes
- the **exact quote** that triggered each flag, so the user can check the reason themselves
- an overall verdict, **RISKY** or **no major red flags**, decided by a fixed, written rule

**User:** a teenager who is about to act on a stock tip from a video.

## Data
- **Source:** [VideoConviction](https://huggingface.co/datasets/gtfintechlab/VideoConviction) (Georgia Tech FinTech Lab, CC BY-NC 4.0): transcripts of YouTube finfluencer videos about US stocks. After removing empty or very short segments: **678 segments from 214 videos**.
- **My labels:** I took **one segment per video** (so no video appears in both sets) and **hand-labeled 70 segments** using my written [labeling guide](LABELING_GUIDE.md): six flags plus an overall RISKY label.
- **RISKY rule:** RISKY = 1 if GUARANTEE, CERTAINTY or ALL_IN is 1, or if URGENCY and NO_RISK are both 1. Promotion alone does not make a video risky.
- **Label review:** after a review I refined the CERTAINTY rule ("warnings and predictions of a fall don't count") and updated 3 labels; every change is logged in the `CHANGE_LOG` column of `labels.csv`.
- **Split:** 20 **dev** videos (3 risky) for improving the prompt, and 50 **test** videos (6 risky) that I did not look at while tuning. The split is stratified with a fixed seed so both sets contain risky videos.

## How it works
- **Model:** Google **Gemini 3.8 Flash** (free tier). The prompt contains my labeling guide word for word, and Gemini returns structured JSON with each flag and its evidence quote. I apply the same RISKY rule to Gemini's flags that I applied to my own labels.
- **Why a pretrained language model?** Spotting hype depends on meaning, not just words: "this could double" (hedged) versus "this WILL double" (certain). I also had far too few labels to train my own model.
- **Working within free-tier limits:** the free tier allows only 20 requests per day, so I sent **5 transcripts per request**, each analyzed independently, and used the same setup for every experiment. The API run of the final test stopped at the daily limit after 30 of 50 videos. I ran the **remaining 20 through the AI Studio website with the same model, prompt and batches**. Note that the website does not allow setting temperature, so those 20 used the default setting instead of temperature 0.

## Baselines
1. **Most common answer:** label every video "not risky."
2. **Keyword rule:** RISKY if the transcript contains words like "guarantee," "10x," "don't miss," "call option" or "short squeeze."

## Improving the prompt (dev set only)
Prompt v1 made **8 flag mistakes** on the 20 dev videos. I found six patterns and added a general rule for each:
1. Business problems (rising costs, competition, debt) count as mentioning risk.
2. "Do your own research" is not a risk warning.
3. Thanking existing members is not promotion; it needs an invitation to the viewer.
4. Judge each flag separately, even when the video is cautious overall.
5. Selling options (for example, selling puts) counts as a risky options strategy.
6. A creator's own buying plans count as a recommendation.

| Dev set (20 videos) | Precision | Recall | F1 | Flag mistakes |
|---|---|---|---|---|
| Gemini, prompt v1 | 1.00 | 0.33 | 0.50 | 8 |
| **Gemini, prompt v2** | 1.00 | 0.67 | 0.80 | **1** |

The v2 rules were written after seeing these dev videos, so the dev improvement is optimistic. The test set below is the fair check. Prompt v2 was locked before running it.

## Results (50 held-out test videos, 6 labeled risky)
| Method | Precision | Recall | F1 | Accuracy |
|---|---|---|---|---|
| Most common answer | 0.00 | 0.00 | 0.00 | 0.88 |
| Keyword rule | 1.00 | 0.83 | 0.91 | 0.98 |
| **FinFlag (Gemini, prompt v2)** | **1.00** | 0.67 | 0.80 | 0.96 |

- FinFlag **caught 4 of 6 risky videos with 0 false alarms**, and agreed with my labels on **292 of 300 individual flag decisions**.
- The first row shows why accuracy is misleading: calling everything "safe" scores 88% accuracy while catching **zero** risky videos.

**Per-flag agreement on test:**
| Flag | Precision | Recall | F1 |
|---|---|---|---|
| NO_RISK | 0.80 | 1.00 | 0.89 |
| ALL_IN | 1.00 | 0.80 | 0.89 |
| PROMOTION | 1.00 | 0.62 | 0.77 |
| CERTAINTY | 0.00 | 0.00 | 0.00 (missed both cases) |
| URGENCY | 0.00 | 0.00 | 0.00 (missed the one case) |
| GUARANTEE | n/a | n/a | n/a (no cases in test) |

### Why the keyword rule scored higher, and what that does and doesn't mean
Most of my risky test videos were risky because of options or short-squeeze trading, and those happen to contain exact phrases a keyword list can match. **The keyword list was also written by my AI assistant after it had read all 70 transcripts, including the test set**, so the baseline had an unfair advantage and its score is likely optimistic. A word list also can't explain *why* a video is risky, and it can't detect a missing risk warning. FinFlag's main value is the evidence quote for every flag, and it raised no false alarms. A fairer comparison would use a keyword list written without seeing the data.

### Where it fails
- **Confident predictions (missed, seg 245):** "The Ethernet market is still growing and will continue to do so for the next decade at least." Gemini didn't treat an industry forecast as a stock prediction. Both CERTAINTY cases were missed.
- **Risky trade framed with heavy risk talk (missed, seg 666):** a short-squeeze setup followed by a long section on risk management. Gemini seems to have been reassured by the warnings.
- **Subtle promotion (segs 45, 600, 165):** creators demonstrating their own software or website without saying "buy." Gemini only flags clear invitations.
- **Urgent selling (seg 184):** "you got to cut losses immediately" is urgency, but it's about protecting money, not FOMO. My own guide is unclear here.

### Fairness check
I compared Gemini's flags with mine for each creator with 3+ videos in my labeled set (12 channels, 3–10 videos each). Mistakes were spread thinly (0–0.4 flag mistakes per video). One channel stood out: I labeled 2 of its 5 videos risky and Gemini flagged none. With so few videos per creator I can't draw firm conclusions, but it's the first place I'd look with more data.

## Limitations
- **Small test set:** only 6 risky test videos, so a single miss changes recall by about 17 percentage points.
- **One labeler:** labels reflect my own judgement using my guide. I did not measure agreement with a second labeler.
- **Mixed run settings:** 20 of the 50 test videos were run through the AI Studio website at default temperature, rather than through the API at temperature 0.
- **Keyword baseline written with knowledge of the data** (see above).
- **Text only and English only:** transcripts miss visual hype such as cars, charts and on-screen captions, and the videos are US-stock YouTube content.

## What I'd do next
- Have two or three people label the same videos and measure how much they agree.
- Label 200+ videos, especially more risky ones, and add TikTok transcripts.
- Write the keyword baseline without looking at the data, for a fair comparison.

## How to run
1. Open `Seeun_PARK_FinFlag.ipynb` in Google Colab (click the "Open in Colab" badge at the top of the notebook).
2. Upload `labels.csv` and `gemini_results_backup.json` to Colab's file panel. The notebook loads all saved Gemini answers automatically, so every result can be reproduced without calling the API.
3. To run new transcripts through the demo, add a Gemini API key as a Colab Secret named `GEMINI_API_KEY`. Never paste the key into the code.

## Sources and AI tools
- VideoConviction dataset, Galarnyk et al., Georgia Tech FinTech Lab: https://huggingface.co/datasets/gtfintechlab/VideoConviction (CC BY-NC 4.0)
- Google Gemini API and Google AI Studio (Gemini 3.8 Flash)
- Libraries: pandas, scikit-learn, Hugging Face `datasets`, Gradio
- **AI assistance:** Claude (Anthropic) helped plan the project and draft the code, the keyword baseline and this README. I labeled all 70 videos myself, designed the red-flag checklist, ran every experiment and reviewed all results.
