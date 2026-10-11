# ODEP & Deep Learning – Label-free stratification of prostate cancer cell lines

**Master's thesis in Medical Engineering** – University of Rome Tor Vergata  
Subject: Pattern Recognition & Machine Learning – A.Y. 2024/2025

- **Candidate:** Silvia Gialloreto
- **Supervisor:** Prof. Eugenio Martinelli
- **Co-supervisors:** Dr. Joanna Filippi, Prof. Arianna Mencattini

> *Caratterizzazione e stratificazione label-free di linee cellulari di carcinoma prostatico mediante tecnologia ODEP e Deep Learning*  
> (Label-free characterisation and stratification of prostate cancer cell lines using ODEP technology and Deep Learning)

---

## Repository contents

| File | Description |
|---|---|
| `Tesi_commissione(STAMPA).pdf` | full thesis (80 pages, in Italian) |
| `PPT_SilviaGialloreto_ODEP.pptx` | slides for the thesis defence (34 slides) |

---

## Summary of the work

The thesis proposes a **label-free**, **lab-on-chip** platform based on **optically induced dielectrophoresis (ODEP)** to discriminate three prostate cancer cell lines of increasing metastatic aggressiveness:

| Line | Origin | Phenotype |
|---|---|---|
| **LNCaP** | lymph node metastasis | androgen-dependent, low invasiveness |
| **DU145** | brain metastasis | castration-resistant |
| **PC3**   | bone metastasis (grade IV) | castration-resistant, highly metastatic |

### The two novelties
1. **Motion along the z axis.** The vertical displacement of the cell induced by the dielectrophoretic field is used instead of the classic xy centroid tracking. It produces a *frequency-dependent defocus signature* that depends on the cell's crossover frequency.
2. **Deep learning directly on frame pixels.** A spatio-temporal **R(2+1)D** network learns cell morphology and defocus dynamics together. This is the first application of deep learning to ODEP technology.

### Device and setup
- Sandwich ODEP chip: ITO glass / microfluidic chamber (60 µm) / photoconductive **hydrogenated amorphous silicon (a-Si:H)** / ITO glass. Projecting a light pattern creates reconfigurable *virtual electrodes*.
- Leica DMi8 inverted microscope (20× objective, 18 MP CMOS camera), digital projector with blue light (460 nm), HP 33120A function generator with a TTi WA301 amplifier.
- Microscope, generator and projector are controlled from a Python GUI.

### Measurement protocol
Two light patterns alternate on the same cell every 2 s:

| Pattern | Frequency | Voltage | Effect |
|---|---|---|---|
| Donut | 25 kHz (fixed, < f<sub>co</sub> ≈ 30 kHz) | 8 Vpp | **nDEP**: the cell rises and goes out of focus |
| Filled circle | 150 kHz down to 30 kHz, 10 kHz steps | 10 Vpp | **pDEP**: the cell is pulled back towards the focal plane |

As the circle frequency approaches the crossover, the pDEP force drops and the cell stays increasingly out of focus. The resulting decreasing envelope is characteristic of each cell line.

---

## Analysis pipeline

**Video pre-processing**
1. Cell segmentation with **Cellpose** and dynamic ROI centring, updated at every stimulation cycle (reliable up to the 8th cycle).
2. Frame-by-frame background normalisation.
3. Intensity stretching to [0, 1] per clip, to preserve the defocus information, and splitting of the videos into clips, one per nDEP/pDEP cycle.

**Branch 1: LDA baseline (scalar defocus only)**
- Michelson contrast per frame, C = (I<sub>max</sub> − I<sub>min</sub>)/(I<sub>max</sub> + I<sub>min</sub>), expressed as the relative change ΔC(t).
- 24 features: 8 descriptors (mean, std, max, min, range, skewness, kurtosis, approximate entropy) computed on ΔC(t) and on its first and second derivatives.
- Forward sequential feature selection, then LDA with 10-fold cross-validation.

**Branch 2: deep learning on frames**
- **2D SqueezeNet** (pre-trained on ImageNet, only conv10 trained, Adam). Input: 8 in-focus frames, one per cycle. This is the morphological baseline.
- **Static R(2+1)D**. Input: the same in-focus frame repeated 8 times. Used as a control test.
- **Dynamic R(2+1)D**. Input: 8 frames obtained by linear interpolation from the 110 frames of the cycle. This is the proposed approach.
- 18-layer R(2+1)D pre-trained on Kinetics-400, transfer learning, SGDR (momentum 0.9, cosine annealing), 64×64×3 input, mini-batches of 64, 50 epochs, data augmentation.
- Cell-independent 80/10/10 split, with the same fixed seed for every configuration.

Dataset: **595 cells** at the 8th cycle, balanced across the three classes.

---

## Results

| Model | Input | Balanced accuracy |
|---|---|---|
| LDA | 24 features from ΔC(t), 8 cycles | 61.7% ± 7.5% |
| 2D SqueezeNet | 8 distinct in-focus frames (morphology) | 78% |
| Static R(2+1)D | 8 identical frames (morphology) | 75% |
| **Dynamic R(2+1)D** | **8 interpolated frames (morphology + dynamics)** | **83%** |

- With LDA, accuracy grows with the number of observed cycles, up to a maximum of 66.2% ± 5.5% at the 11th cycle.
- With the same architecture, going from static (75%) to dynamic R(2+1)D (83%) shows that the ODEP-induced defocus dynamics carry discriminative information that morphology alone does not.
- LNCaP, the hardest line to recognise with the static approaches (42–63%), reaches **86.4%** with the dynamic network. The biologically implausible LNCaP↔PC3 confusion also drops, and the remaining errors are concentrated between PC3 and DU145, which are biologically similar lines.

### Future work
More frames per clip, multiple random seeds to estimate the mean accuracy, experiment-independent validation, architecture optimisation and a larger dataset.

---

For details on the theory (DEP, Clausius-Mossotti factor, shell models), chip fabrication, sample preparation and references, see the thesis PDF.
