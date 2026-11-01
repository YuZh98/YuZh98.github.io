+++
title = 'Fifteen Papers in Modern AI: Reading Notes by a Statistician'
date = 2026-11-01T09:00:00-06:00
draft = false
description = 'Reading notes on fifteen papers behind modern language and image models, from the Transformer to InstructGPT, with the mathematics and a statistical reading of each'
+++

<!--more-->

**Before downloading, note that the notes are put together using LLMs. Claims about each paper were checked against the paper, and my interpretations are marked as such. I welcome corrections.**

> Download link: **[Fifteen Papers: Reading Notes (PDF)](https://github.com/YuZh98/YuZh98.github.io/releases/download/pdfs-v1/posts__Notes-for-FifteenPapers__fifteen-papers-notes.pdf)**

These are my notes on fifteen papers that keep coming up when people discuss modern AI. They are short on purpose. For each paper: the problem, the core mathematics, what was actually shown, and how I read it as a statistician. The PDF has more detail.

Two habits run through the notes.

* **Paper vs. me.** Everything under **Shown** comes from the paper. Everything under **My reading** is my interpretation, and the paper may not say it.
* **Claims stay as strong as the source.** If a paper says "perhaps", so do these notes. Summaries of these papers are often firmer than the papers.

| Thread | Papers | Question |
|---|---|---|
| I. Attention | Transformer, RoPE, ViT, BERT | What does attention compute, and what does it assume? |
| II. Scale | GPT-3, LLaMA, MoE | What does more data or capacity buy? |
| III. Generative models | VAE, GAN, Latent diffusion | How do we fit a distribution we cannot write down? |
| IV. Adaptation | Adapters, LoRA | How little must change to specialize a model? |
| V. Knowledge and preferences | RAG, RLHF, InstructGPT | How do outside knowledge and human judgment enter? |

## I. Attention

### 1. Transformer

*Attention Is All You Need.* Vaswani et al., NeurIPS 2017.

**<u>Problem:</u>** Recurrent sequence models need $O(T)$ sequential steps and pass information along a chain. Remove recurrence and let every token attend directly to every other.

$$\mathrm{Attn}(Q,K,V)=\mathrm{softmax}\big(QK^\top/\sqrt{d\_k}\big)V$$

$$ Q=XW^Q,\\ K=XW^K,\\ V=XW^V.$$

**<u>Shown:</u>**

* Multi-head attention runs this in $h$ subspaces of dimension $d\_k=d/h$. Self-attention costs $O(T^2d)$ per layer with $O(1)$ sequential steps.
* Order enters only through positional encodings; a causal mask gives the decoder its autoregressive structure, conditional on the encoder output.

**<u>My reading:</u>** Row $t$ of the output is $\sum\_j w\_{tj}v\_j$ with $w\_{tj}\propto\exp(q\_t^\top k\_j/\sqrt{d\_k})$: the Nadaraya–Watson weighted-average form, with learned queries, keys and values. This is an analogy of form only. There are no bandwidth asymptotics, and the similarity is not a function of $\\|q-k\\|$ unless norms are fixed.

The $\sqrt{d\_k}$: if the coordinates of $q,k$ are independent with mean 0 and variance 1, then $\mathrm{Var}(q^\top k)=d\_k$. The paper offers this to illustrate a mechanism it only *suspects* (softmax saturation at large $d\_k$), under an assumption not claimed for trained vectors.

{{< figure src="/images/fifteen-papers/fig-01-attention.svg" alt="Mechanism diagram for Transformer" caption="Transformer: mechanism diagram." >}}

### 2. RoPE

*RoFormer: Enhanced Transformer with Rotary Position Embedding.* Su et al., Neurocomputing 2024.

**<u>Problem:</u>** Additive position vectors encode absolute position, though attention usually wants relative position. Find $f\_q,f\_k$ so that position enters the score only through the offset between tokens.

$$f\_q(x\_m,m)=R\_{\Theta,m}W^Qx\_m$$

$$ R\_{\Theta,m}^\top R\_{\Theta,n}=R\_{\Theta,n-m}$$

$$ \theta\_i=10000^{-2(i-1)/d}.$$

**<u>Shown:</u>**

* $R\_{\Theta,m}$ is block-diagonal with $2\times2$ rotations by $m\theta\_i$. The one-parameter group property, not orthogonality alone, gives $\langle f\_q,f\_k\rangle=q\_m^\top R\_{\Theta,n-m}k\_n$.
* The paper derives an upper bound on the score whose position factor shrinks numerically over its plotted range.

**<u>My reading:</u>** Treat coordinate pairs as complex numbers and put $h=n-m$. The score is $\mathrm{Re}\sum\_i\bar q\_ik\_ie^{\mathrm{i}h\theta\_i}$, a trigonometric polynomial in the lag with content-dependent coefficients. For the same vector at both positions this is $\sum\_i|q\_i|^2\cos(h\theta\_i)$, a stationary covariance with spectral measure $\tfrac12\sum\_i|q\_i|^2(\delta\_{\theta\_i}+\delta\_{-\theta\_i})$ (Herglotz, the integer-lag form of Bochner). For $q\neq k$ the lag function is positive semi-definite iff every $\bar q\_ik\_i\ge0$, which generically fails.

Consequence: the score cannot decay to zero. It is almost periodic in $h$. The paper's decay claim concerns an upper bound examined numerically over a plotted range, not the score. This derivation is mine, not the paper's.

{{< figure src="/images/fifteen-papers/fig-14-rope.svg" alt="Mechanism diagram for RoPE" caption="RoPE: mechanism diagram." >}}

### 3. ViT

*An Image is Worth 16x16 Words.* Dosovitskiy et al., ICLR 2021.

**<u>Problem:</u>** Can a plain Transformer replace convolutions? Cut an image into $N=HW/P^2$ patches and treat them as tokens.

$$z\_0=\big[x\_{\mathrm{cls}};\\,x\_p^{(1)}E;\dots;x\_p^{(N)}E\big]+E\_{\mathrm{pos}}.$$

**<u>Shown:</u>**

* Large ViTs on ImageNet-1k alone trail ResNets without strong regularization, reach them with ImageNet-21k, and pass them with 300M-image JFT pretraining, measured after fine-tuning.
* Only the patchwise MLP layers are local and translationally equivariant. 2D structure is injected by hand only at patch extraction and position-embedding interpolation.

**<u>My reading:</u>** A bias–variance reading (mine; the paper's phrase is "large scale training trumps inductive bias"): dropping most of the convolutional bias raises the data requirement. The result is about pretraining scale and is evaluated after transfer, not a statement that CNN priors are wrong.

{{< figure src="/images/fifteen-papers/fig-04-vit.svg" alt="Mechanism diagram for ViT" caption="ViT: mechanism diagram." >}}

### 4. BERT

*Bidirectional Encoder Representations from Transformers.* Devlin et al., NAACL 2019.

**<u>Problem:</u>** Pretrain a bidirectional encoder on unlabeled text. Conditioning on both sides needs an objective other than next-token prediction.

$$\mathcal{L}\_{\mathrm{MLM}}(\theta)=-\mathbb{E}\sum\_{i\in\mathcal{M}}\log p\_\theta\big(x\_i\mid\tilde x\_{1:T}\big).$$

**<u>Shown:</u>**

* 15% of positions are selected; of those, 80% become [MASK], 10% a random token, 10% stay. The paper credits bidirectionality and its two pretraining tasks for most of the gain; pretrain-then-fine-tune already existed.
* Removing next-sentence prediction *hurts* QNLI, MNLI and SQuAD 1.1 in this paper. The finding that it is dispensable is later work, RoBERTa (Liu et al., 2019), with a changed input format.

**<u>My reading:</u>** The objective is composite-likelihood-style (Lindsay, 1988): a sum of conditional log-scores at randomly selected positions, each given the whole corrupted sequence, so it is not Besag's pseudo-likelihood (Besag, 1975). Unlike Lindsay's construction, the terms do not come from one parametric joint model, and the conditionals need not be compatible with any joint density. Hence no tractable generation.

{{< figure src="/images/fifteen-papers/fig-07-bert.svg" alt="Mechanism diagram for BERT" caption="BERT: mechanism diagram." >}}


## II. Scale

### 5. GPT-3

*Language Models are Few-Shot Learners.* Brown et al., NeurIPS 2020.

**<u>Problem:</u>** How far does next-token prediction go at 175B parameters, and does in-context learning, weak in GPT-2, improve with scale?

$$\max\_\theta\sum\_t\log p\_\theta(x\_t\mid x\_{<t})$$

$$\text{few-shot: }p\_\theta\big(\cdot\mid(x\_1,y\_1),\dots,(x\_n,y\_n),x\_{n+1}\big).$$

**<u>Shown:</u>**

* 125M to 175B parameters on about 300B tokens; the largest has $d=12288$ and 96 layers. Few-shot evaluation takes no gradient step.
* Validation loss follows a power law in training compute. The zero-to-few-shot gap *often* grows with capacity, which *perhaps* suggests larger models are more proficient meta-learners.

**<u>My reading:</u>** In-context learning is conditioning under a fixed distribution. The analogy to a posterior predictive is informal only: $\theta$ is a point estimate and there is no latent-task model. The paper's own frame is meta-learning, and it stays explicitly agnostic on whether the model learns a task at inference time or recognizes patterns seen in training.

{{< figure src="/images/fifteen-papers/fig-10-gpt3-icl.svg" alt="Mechanism diagram for GPT-3" caption="GPT-3: mechanism diagram." >}}

### 6. LLaMA

*Open and Efficient Foundation Language Models.* Touvron et al., arXiv 2023.

**<u>Problem:</u>** For a target quality, training a smaller model for longer can lower inference cost. Test this past the usual ratio of tokens to parameters.

**<u>Shown:</u>**

* Sizes 7B to 65B on 1.0 to 1.4T public tokens, with pre-normalization by RMSNorm, SwiGLU, and RoPE.
* 13B outperforms 175B GPT-3 on most benchmarks, as the paper states, with exceptions such as PIQA. Training loss was still falling at the end.

**<u>My reading:</u>** Tokens per parameter from the paper's Table 2: about 149 (6.7B on 1.0T), 77, 43, and 21.5 (65.2B on 1.4T), against the 20:1 baseline the paper quotes ("a 10B model on 200B tokens"). The comparison with GPT-3 is not controlled: architecture and data both differ, and baseline numbers come from the corresponding papers. So it does not isolate the effect of more tokens.

{{< figure src="/images/fifteen-papers/fig-13-llama-block.svg" alt="Mechanism diagram for LLaMA" caption="LLaMA: mechanism diagram." >}}

### 7. MoE

*Outrageously Large Neural Networks.* Shazeer et al., ICLR 2017.

**<u>Problem:</u>** Grow parameters without growing the computation spent on each input, by routing each input to a few experts.

$$y=\sum\_{i=1}^EG(x)\_ie\_i(x)$$

$$ G(x)=\mathrm{softmax}\big(\mathrm{KeepTop}k(H(x))\big).$$

**<u>Shown:</u>**

* Expert FLOPs scale with $k$, not $E$. The tunable noise in $H$ helps load balancing and supplies the smoothness that lets gradients pass through.
* Language models reached 131,072 experts (137B), but the *best* perplexity was at 65,536 (68B); 131,072 degraded. Translation used up to 2048 experts.

**<u>My reading:</u>** A covariate-dependent sparse combination in the Jordan–Jacobs mixture-of-experts tradition, which the paper cites for softmax gating. The form $\sum\_i g\_i\mu\_i$ matches a mixture-of-experts conditional mean; what is absent is a mixture likelihood or latent indicator. The layer is a network component trained by backpropagation. The load losses are hand-weighted squared coefficients of variation across experts in a batch.

{{< figure src="/images/fifteen-papers/fig-11-moe.svg" alt="Mechanism diagram for MoE" caption="MoE: mechanism diagram." >}}


## III. Generative models

### 8. VAE

*Auto-Encoding Variational Bayes.* Kingma and Welling, ICLR 2014.

**<u>Problem:</u>** Fit $p\_\theta(x,z)$ with continuous latents when the posterior is intractable and the data are large.

$$\log p\_\theta(x)\ge\mathbb{E}\_{q\_\phi(z\mid x)}[\log p\_\theta(x\mid z)]-\mathrm{KL}\big(q\_\phi(z\mid x)\\,\\|\\,p(z)\big)$$

$$ z=\mu\_\phi(x)+\sigma\_\phi(x)\odot\varepsilon.$$

**<u>Shown:</u>**

* Reparameterization turns the gradient into a pathwise expectation, replacing a naive estimator the paper reports as very high variance. The Gaussian KL is closed form.

**<u>My reading:</u>** Amortized variational inference: one network $q\_\phi$ replaces per-datum variational parameters. ("Amortized" is the later word; the paper says recognition model.) Then $\log p\_\theta(x)-\mathcal{L}=\mathrm{KL}(q\_\phi(z\mid x)\\,\\|\\,p\_\theta(z\mid x))$ exactly, so the bound is tight iff the approximate posterior is exact.

The link to diffusion is narrower than often said: both use $z=\text{mean}+\text{scale}\cdot\varepsilon$, but diffusion's forward process has no trainable parameters, so no pathwise gradient through sampling is needed.

{{< figure src="/images/fifteen-papers/fig-05-vae.svg" alt="Mechanism diagram for VAE" caption="VAE: mechanism diagram." >}}

### 9. GAN

*Generative Adversarial Nets.* Goodfellow et al., NeurIPS 2014.

**<u>Problem:</u>** Learn to sample without writing a likelihood: train a generator against a discriminator.

$$\min\_G\max\_D\\ \mathbb{E}\_{p\_{\mathrm{data}}}[\log D(x)]+\mathbb{E}\_{p\_z}[\log(1-D(G(z)))]$$

$$ D^\ast=\frac{p\_{\mathrm{data}}}{p\_{\mathrm{data}}+p\_g}.$$

**<u>Shown:</u>**

* With optimal $D$ the objective is $-\log4+2\\,\mathrm{JSD}(p\_{\mathrm{data}}\\,\\|\\,p\_g)$. Training alternates updates, and the generator in practice maximizes $\log D(G(z))$ to avoid saturation.
* The paper warns of "the Helvetica scenario", later called mode collapse, and claims *no* sample-quality advantage over existing methods.

**<u>My reading:</u>** Where $p\_{\mathrm{data}},p\_g>0$, with equal class weights and at the optimum, $D^\ast/(1-D^\ast)=p\_{\mathrm{data}}/p\_g$, so $\mathrm{logit}\\,D^\ast$ is the log density ratio: the discriminator is a classifier-based density-ratio estimator. The paper treats $D$ as maximum likelihood for $P(Y=y\mid x)$, not in these words.

The JSD statement is a non-parametric result. The paper says so itself: optimizing $\theta\_g$ rather than $p\_g$ means "the proofs do not apply".

{{< figure src="/images/fifteen-papers/fig-06-gan.svg" alt="Mechanism diagram for GAN" caption="GAN: mechanism diagram." >}}

### 10. Latent diffusion

*High-Resolution Image Synthesis with Latent Diffusion Models.* Rombach et al., CVPR 2022.

**<u>Problem:</u>** Pixel-space diffusion is expensive. Train a perceptual autoencoder, then run diffusion on the latent, with conditioning by cross-attention.

$$\mathcal{L}\_{\mathrm{LDM}}=\mathbb{E}\\,\big\\|\varepsilon-\varepsilon\_\theta\big(z\_t,t,\tau(y)\big)\big\\|^2$$

$$ z\_t=\sqrt{\bar\alpha\_t}\\,z\_0+\sqrt{1-\bar\alpha\_t}\\,\varepsilon.$$

**<u>Shown:</u>**

* Downsampling factors 4 and 8 work best. Savings are empirical, not a scaling law: 916 V100-days for ADM against 271 for LDM-4 and 79 for LDM-8 on ImageNet.
* The extracted paper text never mentions Stable Diffusion.

**<u>My reading:</u>** Denoising is score estimation. Under squared loss the optimum is $\varepsilon^\ast(z\_t,t)=\mathbb{E}[\varepsilon\mid z\_t]$, and since $z\_t\mid z\_0\sim\mathcal{N}(\sqrt{\bar\alpha\_t}z\_0,(1-\bar\alpha\_t)I)$, Tweedie's formula (Efron, 2011) gives $\nabla\log q\_t(z\_t)=-\mathbb{E}[\varepsilon\mid z\_t]/\sqrt{1-\bar\alpha\_t}$. The paper says only that its objective mirrors denoising score matching.

{{< figure src="/images/fifteen-papers/fig-08-latent-diffusion.svg" alt="Mechanism diagram for Latent diffusion" caption="Latent diffusion: mechanism diagram." >}}


## IV. Adaptation

### 11. Adapters

*Parameter-Efficient Transfer Learning for NLP.* Houlsby et al., ICML 2019.

**<u>Problem:</u>** Share one frozen backbone across tasks by inserting small trainable modules.

$$\mathrm{Adapter}(h)=h+W\_{\mathrm{up}}\\,g(W\_{\mathrm{down}}h)$$

$$ m\ll d.$$

**<u>Shown:</u>**

* Two adapters per Transformer layer; adapters, layer norms and the task head train.
* GLUE on BERT-large: 80.0 against 80.4 for full fine-tuning with about 3.6% of parameters and a per-task adapter size.

**<u>My reading:</u>** Adapters are a nonlinear residual bottleneck on *activations*; LoRA is linear in $x$ and bilinear in $(B,A)$, a reparameterization of a *weight*, so it can merge away. Both start at or near the pretrained model: approximately for adapters, exactly for LoRA with $B=0$.

{{< figure src="/images/fifteen-papers/fig-03-adapter.svg" alt="Mechanism diagram for Adapters" caption="Adapters: mechanism diagram." >}}

### 12. LoRA

*Low-Rank Adaptation of Large Language Models.* Hu et al., ICLR 2022.

**<u>Problem:</u>** Fine-tuning large models is costly in memory and storage. Freeze $W\_0$ and learn a low-rank update.

$$h=W\_0x+\frac{\alpha}{r}BAx$$

$$ B\in\mathbb{R}^{d\_{\mathrm{out}}\times r},\\ A\in\mathbb{R}^{r\times d\_{\mathrm{in}}}$$

$$ B\_0=0.$$

**<u>Shown:</u>**

* On the paper's three GPT-3 175B tasks, $r$ as small as 1 to 4 on $W^Q,W^V$ matches full fine-tuning at about 0.003 to 0.01% of parameters. The authors do not expect this for every task.
* Evidence for low intrinsic rank is indirect: accuracy is nearly flat in $r$, and only the top singular direction of learned $A$ is shared between $r=8$ and $r=64$.

**<u>My reading:</u>** A reduced-rank parameterization of the update, in the spirit of reduced-rank regression (Izenman, 1975): $\mathrm{rank}(\Delta W)\le r$. When $B,A$ have full rank $r$, the factorization is identified only up to $(B,A)\mapsto(BG,G^{-1}A)$ for $G\in GL(r)$, so $\Delta W$ has $r(d\_{\mathrm{in}}+d\_{\mathrm{out}})-r^2$ free dimensions, not $r(d\_{\mathrm{in}}+d\_{\mathrm{out}})$. The paper does not state this.

The initialization is asymmetric for a reason the paper leaves implicit: $A=B=0$ is a stationary point, while at $B=0$ with $A\neq 0$ we have $\partial L/\partial A=0$ and $\partial L/\partial B\neq0$ in general, so $B$ moves first and both train after that.

{{< figure src="/images/fifteen-papers/fig-02-lora.svg" alt="Mechanism diagram for LoRA" caption="LoRA: mechanism diagram." >}}


## V. Knowledge and preferences

### 13. RAG

*Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.* Lewis et al., NeurIPS 2020.

**<u>Problem:</u>** Combine a parametric generator with a non-parametric document memory, and train through the retrieval.

$$\text{Sequence: }p(y\mid x)\approx\sum\_{z\in\mathrm{top}\text{-}k}p\_\eta(z\mid x)\prod\_ip\_\theta(y\_i\mid x,z,y\_{<i}),$$ $$\text{Token: }p(y\mid x)\approx\prod\_i\sum\_{z}p\_\eta(z\mid x)\\,p\_\theta(y\_i\mid x,z,y\_{<i}).$$

**<u>Shown:</u>**

* The query encoder and BART generator train end to end; the passage encoder and index stay fixed. Fine-tuning uses no passage labels, though the retriever starts from supervised DPR.
* The generator runs once per passage. The latent-variable idea existed for extractive QA; the contribution is extending it to general sequence-to-sequence generation.

**<u>My reading:</u>** RAG-Sequence is a finite mixture over retrieved passages with latent $z$, fitted by marginal likelihood with Adam, not EM. The components depend on $x$ through the top-$k$ set, and truncation is the stated approximation. The weights sum to one only if renormalized: the paper does not settle this, though the Hugging Face implementation does renormalize.

RAG-Token is exactly the marginal under $z\_i\overset{\mathrm{iid}}{\sim}p\_\eta(\cdot\mid x)$, one latent per token, since $\sum\_{z\_{1:N}}\prod\_i=\prod\_i\sum\_{z\_i}$. The two models coincide for length-one outputs.

{{< figure src="/images/fifteen-papers/fig-09-rag.svg" alt="Mechanism diagram for RAG" caption="RAG: mechanism diagram." >}}

### 14. RLHF

*Deep Reinforcement Learning from Human Preferences.* Christiano et al., NeurIPS 2017.

**<u>Problem:</u>** When no reward can be written down, learn one from human comparisons of behavior clips. The approach is older; the contribution is scaling it to deep RL.

$$\hat P[\sigma^1\succ\sigma^2]=\frac{\exp\sum\_t\hat r(s^1\_t,a^1\_t)}{\exp\sum\_t\hat r(s^1\_t,a^1\_t)+\exp\sum\_t\hat r(s^2\_t,a^2\_t)}.$$

**<u>Shown:</u>**

* Policy training, human labeling and reward fitting run concurrently, with cross-entropy on the labels.
* About 900 queries, under an hour of the authors' feedback, taught a simulated Hopper a backflip.

**<u>My reading:</u>** Bradley–Terry (Bradley and Terry, 1952) with log-strength equal to summed reward, $P(\sigma^1\succ\sigma^2)=\sigma(R\_1-R\_2)$. The fitted likelihood is the contaminated version $0.05+0.9\\,\sigma(R\_1-R\_2)$.

The likelihood sees $\hat r$ only through differences of segment sums, so a constant shift cancels for equal-length segments; further invariances can exist, and $\hat r$ is constrained only on visited state–action pairs. The scale is relative to human choice noise, which the paper treats as arbitrary, normalizing $\hat r$ to a fixed standard deviation. Query selection by ensemble disagreement is similar in spirit to query-by-committee; the paper calls it crude and reports that it hurts on some tasks.

{{< figure src="/images/fifteen-papers/fig-12-rlhf.svg" alt="Mechanism diagram for RLHF" caption="RLHF: mechanism diagram." >}}

### 15. InstructGPT

*Training Language Models to Follow Instructions with Human Feedback.* Ouyang et al., NeurIPS 2022.

**<u>Problem:</u>** GPT-3 continues text rather than following instructions. Align it by supervised fine-tuning, then a reward model on human rankings, then PPO.

$$L(\psi)=-\mathbb{E}\big[\log\sigma\big(r\_\psi(x,y\_w)-r\_\psi(x,y\_l)\big)\big],$$ $$\max\_\theta\\ \mathbb{E}\Big[r\_\psi(x,y)-\beta\log\frac{\pi\_\theta(y\mid x)}{\pi^{\mathrm{SFT}}(y\mid x)}\Big]+\gamma\\,\mathbb{E}\big[\log\pi\_\theta(x)\big].$$

**<u>Shown:</u>**

* Raters preferred 1.3B InstructGPT to 175B GPT-3. Truthfulness improved overall, except that the 1.3B PPO-ptx model did slightly worse than a same-size GPT-3 model.
* The $\gamma$ term reduces, but does not remove, the alignment tax. Residual failures include false premises, over-hedging and multiple constraints.

**<u>My reading:</u>** The reward loss is logistic log-loss in the reward difference. That is Bradley–Terry in form, but the label is mine: the paper never names it. Pairs from one ranking are correlated, as the paper notes, so the sum is composite-likelihood-like rather than a ranking likelihood.

With $\gamma=0$ and $r$ fixed, $\max\_\pi\mathbb{E}\_\pi[r]-\beta\\,\mathrm{KL}(\pi\\,\\|\\,\pi^{\mathrm{SFT}})$ over all conditionals is solved pointwise in $x$ by the exponential tilt $\pi^\ast(y\mid x)\propto\pi^{\mathrm{SFT}}(y\mid x)\exp(r(x,y)/\beta)$, the Gibbs variational principle. The paper does not state this; a parametric policy under PPO only approximates it. So $\beta$ is a temperature.

{{< figure src="/images/fifteen-papers/fig-15-instructgpt.svg" alt="Mechanism diagram for InstructGPT" caption="InstructGPT: mechanism diagram." >}}

## Threads

**<u>Where old statistics reappears:</u>**

* Paired comparisons: Bradley–Terry (1952) in RLHF, and in form in InstructGPT.
* Latent-variable marginal likelihood in RAG; amortized variational inference in VAE.
* Density-ratio estimation in GAN; score estimation through Tweedie in diffusion.
* Exponential tilting in the KL-penalized objective of InstructGPT.
* A spectral representation in RoPE, the reading closest to spatial statistics.

**<u>Where this lens stops:</u>** GPT-3 and LLaMA are mainly empirical findings about scale and engineering. I see no theorem underneath them, and these notes do not pretend otherwise.

**<u>What reading the source changed:</u>**

* RoPE's score does not decay. A bound does, numerically, over a plotted range.
* BERT's own experiments say next-sentence prediction helps.
* The largest MoE model was not the best one.
* Latent diffusion reports empirical savings, not a scaling law.
* GPT-3 says "often" and "perhaps" where summaries say "shows".

## References for the statistical readings

* Bradley, R. A. and Terry, M. E. (1952). Rank analysis of incomplete block designs: I. The method of paired comparisons. *Biometrika* 39, 324–345.
* Besag, J. (1975). Statistical analysis of non-lattice data. *The Statistician* 24, 179–195.
* Efron, B. (2011). Tweedie's formula and selection bias. *JASA* 106, 1602–1614.
* Izenman, A. J. (1975). Reduced-rank regression for the multivariate linear model. *Journal of Multivariate Analysis* 5, 248–264.
* Lindsay, B. G. (1988). Composite likelihood methods. *Contemporary Mathematics* 80, 221–239.
