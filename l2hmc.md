---
title: "l2hmc-qcd -- DWQ @ 25"
theme: simple
width: 960px
height: 700px
center: false
margin: 0.04
highlightTheme: github
transition: slide
output:
  revealjs::revealjs_presentation:
    self_contained: false
    reveal_plugins: ["chalkboard"]
    reveal_options:
      chalkboard:
        enabled: true
        theme: whiteboard
        toggleNotesButton: false
---

<!-- .slide: data-background="#1c1c1c" -->

<div id='dark'>

&nbsp;

### Accelerated Sampling Techniques
#### in Lattice Gauge Theory

[BNL &amp; RBRC: "DWQ @ 25"](https://indico.bnl.gov/event/13576/)
<br>
December, 2021
<br>
<br>

[**Sam Foreman**](https://www.samforeman.me)

<br>

<small>[1] [arXiv: 2105.03418](https://arxiv.org/abs/2105.03418), 
   [2] [arXiv: 2112.01582](https://arxiv.org/abs/2112.01582)
 [3] [arXiv: 2112.01586](https://arxiv.org/abs/2112.01586)</small>
</small>

[<img align="left" width=10% src="assets/github.svg">](https://github.com/saforem2/l2hmc-qcd)

[<img align="right" width=30% src="assets/Argonne_cmyk_white.svg">](https://alcf.anl.gov)

</div>

<!--
# Introduction

<div id="left" style="width=50%;">

- **Standard Model:**
    - E&amp;M, strong, weak interactions, elementary particles

- **Quantum Chromodynamics (QCD)**:
	- Theory of the strong interaction between quarks and gluons
	- **Analytically intractable**
        - Discretize space-time onto lattice

</div>

<div id="right" style="width=50%;">


<img class="imgnoborder" src="assets/nucleus.svg" style="width:200px;" />

<br>

<img class="imgnoborder" src="assets/feynman.svg" style="width:450px;" />


</div>

-->

---
<!-- .slide: data-background="#1c1c1c" -->

<div id='dark'>


## Motivation

- Want to calculate observables
  $$ \langle \mathcal{O}\rangle\propto\int\left[\mathcal{D}x\right]\mathcal{O}(x)e^{-S(x)} $$
- If we had _independent configurations_, we could approximate the integral as
  $$ \langle\mathcal{O}\rangle\simeq\frac{1}{N}\sum_{n=1}^{N}\mathcal{O}(x_{n})\Rightarrow \sigma^{2}=\frac{1}{N}\text{Var}\left[\mathcal{O}(x)\right] $$

---
<!-- .slide: data-background="#1c1c1c" -->

<div id='dark'>

## Motivation

<span style="font-size:0.8em;">

- For independent samples: 
  $$\langle \mathcal{O}\rangle \propto\int\left[\mathcal{D}x\right]\mathcal{O}(x)e^{-S(x)}
  \simeq\frac{1}{N}\sum_{n=1}^{N}\mathcal{O}(x_{n})$$
  $$\Rightarrow \sigma^{2}=\frac{1}{N}\text{Var}\left[\mathcal{O}(x\right)]$$
- Accounting for <span id="blue">autocorrelations</span>:
  $$ \sigma^{2}=\frac{\color{#228BE6}{\tau_{\mathrm{int}}^{\mathcal{O}}}}{N}\text{Var}\left[\mathcal{O}(x)\right] $$
- <span id="blue">$\tau_{\mathrm{int}}^{\mathcal{O}}$</span> is known to scale <span
  id="red">exponentially</span> as we approach physical lattice spacing.

</span>


---
<!-- .slide: data-background="#1c1c1c" -->

<div id='dark'>

## Motivation

<!-- - The ability to efficiently sample from complicated sampling from complicated distributions is a widely studied  -->
- Generating independent configurations is currently a major bottleneck for
  lattice QCD.

- As the lattice spacing $a\rightarrow 0$ (or equivalently, $\beta \rightarrow \infty$),
  configurations get stuck in sectors of fixed gauge topology. 

  - Causes $\tau_{\mathrm{int}}$ to grow exponentially

  ![](assets/single_chain.svg)

---

# <span style="color: #3B4CC0;">Critical Slowing Down</span>

<div class="row">

<div class="column" style="width: 40%; font-size: 85%;">

- Generating independent configurations is currently a major bottleneck for lattice QCD.

- As $\beta\rightarrow \infty$, configurations get stuck in sectors of fixed gauge topology $Q = \text{ const. }$ 

  - $\Rightarrow$ \# of configurations required to reliably estimate errors <span id="red">**increases exponentially**</span> 
  - $\tau_{\mathrm{int}} \rightarrow \infty$

</div>

<div class="column" style="width: 59%;">

<img src="assets/critical_slowing_down.svg" style="max-width:85%; height:auto;">

</div>

</div>

---
<!-- .slide: data-background="#1c1c1c" -->

<div id='dark'>

### HMC: Leapfrog Integrator

![](assets/hmc1.svg)  <!-- .element width="70%" -->

<iframe data-src="https://chi-feng.github.io/mcmc-demo/app.html"></iframe> <!-- .element width="95%" height="300px" -->

---

# Issues with HMC
	
- Energy levels selected randomly $\rightarrow$ slow mixing!
- Cannot easily traverse low-density zones
- What do we want in a good sampler?
  - **Fast mixing** (small autocorrelations) 
  - **Fast burn-in** (quick convergence)
  - Ability to mix across energy levels and isolated modes

![](assets/hmc_traj_eps05.svg) <!-- .element width="49%" -->
![](assets/hmc_traj_eps025.svg) <!-- .element width="49%" -->

---

# Leapfrog Layer

<div id="left" style="width:40%; align:center; font-size:0.9em; text-align: left;">

- <span id="red"><b><u>L2HMC Update</u></b></span>:

  ![](assets/updates.svg)

</div>

&nbsp;

![](assets/leapfrog_layer.svg) <!-- .element width="55%" align="center" -->

![](assets/network_functions.svg)


---
<!-- .slide: data-background="#1c1c1c" -->

<div id='dark'>

## Leapfrog Layer

<div id="left" style="width:40%; align:center; font-size:0.66em; text-align: left;">

### <u>L2HMC Update</u>:
1. Update $\mathbf{v}$:

    - `\(\mathbf{v}'= \Gamma^{\pm}[\mathbf{v}; \zeta_{\mathbf{v}}]\)`

2. Update <span id="red">half</span> of `\(\mathbf{x}\)` via <span id="red"> `\(\mathbf{x}_{\bar{m}}\)`</span>:

    - `\(\mathbf{x}' = \)` <span id="blue">`\(\mathbf{x}_{m} \)`</span>`\(+ \Lambda^{\pm}[\)`<span id="red">`\(\mathbf{x}_{\bar{m}}\)`</span>`\(; \zeta_{\mathbf{x}}]\)`

3. Update <span id="blue">(other) half</span> of `\(\mathbf{x}\)` via <span id="blue"> `\(\mathbf{x}_{k}'\)`</span> :

    - `\(\mathbf{x}'' = \)` <span id="red">`\(\mathbf{x}_{\bar{m}} \)`</span>`\(+ \Lambda^{\pm}[\)`<span id="blue">`\(\mathbf{x}_{m}'\)`</span>`\(; \zeta_{\mathbf{x}'}]\)`

4. Update $\mathbf{v}$:

    - `\(\mathbf{v}''= \Gamma^{\pm}[\mathbf{v}'; \zeta_{\mathbf{v}'}]\)`

</div>

<br>

![](assets/leapfrog_layer_dark2.svg) <!-- .element width="58%" -->
![](assets/network_functions_dark.drawio.svg) <!-- .element width="100%" align="center" -->



---

## Toy Example: GMM $\in \mathbb{R}^{2}$

![](assets/iso_gmm_chains.svg)

---

# Lattice Gauge Theory
	
<div id="left" class="float:left" style="padding-left:20px; text-align:left; max-width=30%;">

- <b>Link variables</b>: 
  $\color{#228BE6}{U_{\mu}(x) = e^{i x_{\mu}(n)}\in U(1)}$

  with <span style="font-size:0.9em;">$x_{\mu}(n)\in[-\pi,\pi]$</span>


- <b>Wilson Action</b>: $\color{#228BE6}{S_{\beta}(x) = \beta\sum_{P} 1 - \cos x_{P}}$,
	
  <span style="font-size:0.8em;">$x_{P}= x_{\mu}(n) + x_{\nu}(n+\hat{\mu})-x_{\mu}(n+\hat{\nu})-x_{\nu}(n)$</span>
	
</div>
	
<div id="right" style="width=40%;">

![](assets/plaq_tikz.svg) <!-- .element align="right" width="70%" -->

</div>

<div class="float:left" style="padding-left:20px; width=100%; text-align:left;">

- <b>Topological Charge</b>:

  <span id="note" style="padding:10px;background:#D0F3D5;">✅ $Q_{\mathbb{R}} = \frac{1}{2\pi}\sum_{P} \sin x_{P}\in\mathbb{R}$ </span> 

  <span id="note" style="padding:10px;background:#F7C2CC;">❌ $Q_{\mathbb{Z}} = \frac{1}{2\pi}\sum_{P} \left\lfloor x_{P}\right\rfloor\in\mathbb{Z}$ 
	
  here $\left\lfloor x_{P}\right\rfloor = x_{P}-2\pi\left\lfloor\frac{x_{P}+\pi}{2\pi}\right\rfloor$

---
### Integrated Autocorrelation Time: $\tau_{\mathrm{int}}$
![](assets/autocorr_new.svg) <!-- .element width="50%" -->
![](assets/charge_histories.svg) <!-- .element width="90%" -->
	  
---

# Interpretation

![](assets/ridgeplots.svg)
<br>
<span style="font-size:0.6em; text-align: center;">
<b>(a.)</b> Deviation in the average plaquette ; 
<b>(b.)</b> Real-valued topological charge ; 
<b>(c.)</b> Effective energy ;
</span>

<br>
<b>Fig.</b> Illustration of how different observables evolve over a
single L2HMC trajectory.
</small>

---

<!-- .slide: data-background="#1c1c1c" -->

# [<img width=8% src="assets/github.svg" align="top">](https://github.com/saforem2/l2hmc-qcd) [l2hmc-qcd](https://github.com/saforem2/l2hmc-qcd)

- [arXiV:2105.03418](https://arxiv.org/abs/2105.03418)

- Source code publicly available

- Both `pytorch` and `tensorflow` implementations with support for distributed training, automatic checkpointing, etc.

- Generic interface, easily extensible

- <b>Work in progress</b> scaling up to 2D, 4D $SU(3)$

---

## Non-Compact Projection 
<small>[arXiv:2002.02428](https://arxiv.org/abs/2002.02428)</small>

<div style="font-size:0.8em;">

- Project $x \in[-\pi, \pi]$ onto $\mathbb{R}$ using a transformation $z = g(x)$:
  $$ z = \tan\left(\frac{x}{2}\right) $$
- Perform the update in $\mathbb{R}$:
  $$ z' = m^{t}\odot z + \bar{m}^{t}\odot \left[\alpha z + \beta\right]$$
- Project back to $[-\pi, \pi]$ using $x = g^{-1}(z)$:
  $$ x = 2 \tan^{-1}(z) $$

</div>

---

# Non-Compact Projection

- Combine into a single update:
  $$ x' = \color{#228BE6}{m^{t}}\odot x +
  \color{#FA5252}{\bar{m}^{t}}\odot\left[2\tan^{-1}\left(\alpha\tan\left(\frac{x}{2}\right)\right)+\beta\right]
  $$
- With corresponding Jacobian:
  $$ \frac{\partial x'}{\partial x} = \frac{\exp(\varepsilon s_{x})}{\cos^{2}(x/2)+exp(2\varepsilon s_{x})\sin(x/2)} $$


</div>

---

## Acknowledgements

<div id="left">

### Collaborators:
 - Xiao-Yong Jin
 - James C. Osborn

### References:
 - [Link to slides](https://bit.ly/l2hmc-ect2021)
 - [Link to github](https://github.com/saforem2/l2hmc-qcd)
 - [reach out!](mailto://foremans@anl.gov)
 - [Link to HMC demo](https://chi-feng.github.io/mcmc-demo/app.html)
 - [arXiv:2105.03418](https://arxiv.org/abs/2002.02428)
 - [arXiv:2002.02428](https://arxiv.org/abs/2002.02428)


</div>

<div id="right">

### Huge thank you to:
 - Yannick Meurice
 - Norman Christ
 - Akio Tomiya
 - Luchang Jin
 - Chulwoo Jung
 - Peter Boyle
 - Taku Izubuchi
 - Critical Slowing Down group (ECP)
 - ALCF Staff + Datascience Group

</div>

<small> 
This research used resources of the Argonne Leadership Computing Facility,
which is a DOE Office of Science User Facility supported under Contract
DE-AC02-06CH11357.
</small>

---

### Network Architectures

<img src="assets/dynamics_xnet0.png"  align=center width=45%>

---

### Network Architectures

<img src="assets/dynamics_vnet.png"  align=center width=36%>

---
<style>

:root {
    --r-heading-text-transform: none;
    --r-block-margin: 20px;
    --r-heading-margin: 0 0 20px 0;
    --r-heading-font: "Open Sans", Helvetica, Impact, sans-serif;
    --r-main-font: 'Source Sans Pro', Helvetica Neue, Helvetica, Arial, sans-serif;
    --r-main-font-size: 38px;
    --r-block-margin: 10px;
    --r-heading-margin: 0 0 20px 0;
    --r-heading-line-height: 1.2;
    --r-heading-letter-spacing: -0.45px;
    --r-heading-word-spacing: 0.5px;
    --r-heading-text-transform: none;
    --r-heading-text-shadow: none;
    --r-heading-font-weight: 800;
    --r-heading1-text-shadow: none;
    --r-heading1-size: 2.0em;
    --r-heading2-size: 1.5em;
    --r-heading3-size: 1.20em;
    --r-heading4-size: 1.15em;
    --r-heading5-size: 1.10em;
    --r-heading6-size: 1.05em;
    --r-code-font: "agave Nerd Font", monospace;
    --r-link-color: #007DFF;
    --r-link-color-dark: #f92672;
    --r-link-color-hover: #63ff51;
    --r-controls-color: #228BE6;
    --r-progress-color: #00CCFF;
    --r-selection-background-color: rgba(30, 60, 107, 0.9);
    --r-selection-color: #fff;
    --r-main-color: #222;
    --r-heading-color: #222;
    --r-background-color: #fff;
}

.reveal {
    font-family: var(--r-main-font), sans-serif;
    font-size: var(--r-main-font-size);
    font-weight: normal;
    color: var(--r-main-color);
}

.reveal h1,
.reveal h2,
.reveal h3,
.reveal h4 {
    margin: var(--r-heading-margin);
    color: var(--r-heading-color);
    font-family: var(--r-heading-font);
    font-weight: 800;
    line-height: var(--r-heading-line-height);
    letter-spacing: var(--r-heading-letter-spacing);
    word-spacing: var(--r-heading-word-spacing);
    text-transform: var(--r-heading-text-transform);
    text-shadow: var(--r-heading-text-shadow);
    word-wrap: break-word;
}

.reveal h1 {
    font-size: var(--r-heading1-size);
}

.reveal h2 {
    font-size: var(--r-heading2-size);
}

.reveal h3 {
    font-size: var(--r-heading3-size);
    color: #1c1c1c;
}

.reveal h4 {
    font-size: var(--r-heading4-size);
    color: #333333;
}

#left {
  margin: 0 0 5px 5px;
  text-align: left;
  float: left;
  z-index: -10;
  width: 48%;
  font-size: 0.85em;
}

#right {
  margin: 0 0 5px 0;
  float: right;
  max-width: 48%;
  text-align: left;
  z-index: -10;
  width: 48%;
  font-size: 0.85em;
}
#dark_back {
    background-color: #1c1c1c;
    color: #efefef;
    .reveal a {
        color: #F92672;
        transition: color 0.15s ease;
    }
    .reveal a:hover {
        color: var(--r-link-color-hover);
    }
}
#blue {
    color: #228BE6;
}
#bright {
    color: #00A2FF;
}
#green {
    color: #009051;
}
#lighpink {
    color: #E64980;
}
#pink {
    color: #F92672;
}

#red {
    color: #FA5252;
}

#noteinverse {
    background-color: #1c1c1c;
    border-radius: 5px;
    border-color: #1c1c1c;
    color: #efefef;
    padding: auto;
    margin: auto;
}

#note {
    background-color: rgba(240, 240, 240, 0.90);
    border-radius: 5px;
    border-color: rgba(240, 240, 240, 1.0);
    padding: auto;
    margin: auto;
}

#halfsize {
    font-size: 0.5em;
}

.container {
  position: relative;
}

.bottomright {
  position: absolute;
  bottom: 8px;
  right: 16px;
  font-size: 18px;
}

/* Responsive layout - makes the menu and the content (inside the section) sit on top of each other instead of next to each other */
@media (max-width: 600px) {
  section {
    -webkit-flex-direction: column;
    flex-direction: column;
  }
}

.row {
  display: flex;
}

.column {
  flex: 50%;
}

</style>
