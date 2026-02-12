A complete end-to-end reasoning walkthrough.

The purpose of this entire workflow is:

To understand which cell–cell communication edges in the tumor microenvironment (TME) control whether a patient responds to therapy, and to simulate how drugs would change those edges — so we can predict which drugs push a patient from “non-responder” to “responder.”

This is a TME network steering problem, not a classical gene-based biomarker problem.

Here’s the entire process, step-by-step.

STEP 1 — Start With Bulk RNA-seq as the Only Input

Bulk RNA-seq is the simplest and most widely available data type.

But it gives you only one mixture signal per patient.

Biologically, however:

responders vs non-responders differ in cell-type composition
cell types differ in their internal expression programs
response is driven by interactions between cell types (cell-cell communication)
not just what is happening inside tumor cells
Therefore, we need to unmix the bulk signal into biologically meaningful units.

STEP 2 — Deconvolution: Split Bulk Data Into Cell Types

We use a technique like BayesPrism or InstaPrism to answer two key questions:

2.1 How much of each cell type is present in each patient?
(e.g., CD8 T cells, NK cells, macrophages, CAFs)

2.2 What is the expression profile of each cell type inside each patient?

This gives us two core ingredients:

A cell fraction matrix:
sample × cell type
A cell-type–specific gene expression matrix:
sample × cell type × gene
Now each patient is represented as a structured multicellular ecosystem, not one giant mixture.

STEP 3 — Build the Cell–Cell Communication Network

Response to immunotherapy is driven by communication between cell types:

Tumor cells present antigens to T cells
Macrophages suppress T cells
CAFs remodel the matrix
B cells secrete cytokines
etc.

Example:

CD8_Tcell (IFNG)  →  Tumor (IFNGR1)

CAF (CXCL12)      →  T cell (CXCR4)

This creates a TME communication graph for each patient.

This is the data structure we will learn from.

STEP 4 — Identify TSFs: The “Drivers” of Response

From hundreds of possible LR edges, we want to find:

Which cell–cell communication edges explain the difference between responders vs non-responders?

We use L1-penalized logistic regression (a sparse model):

It selects a small number of highly predictive edges
These are called TSFs (Tumor Steering Factors)
TSFs are the small set of communication edges that best summarize what makes a patient a responder.

Examples might include:

CD8 → Tumor via IFNG–IFNGR1
NK → Macrophage via TNF–TNFRSF1A
CAF → T cell via CXCL12–CXCR4
These TSFs become the axes of the patient’s TME state.

STEP 5 — Convert TSFs to a Probability Distribution

Why?

Because next we want to do information geometry, which works on probability distributions.

For each patient, we take their TSF edge strengths and compute a normalized “TSF profile”:

Now each patient sits at a point on a probability simplex.

This lets us measure similarity/dissimilarity in a mathematically principled way.

STEP 6 — Information Geometry: Define the “Responder Manifold”

We want to treat responders as defining the “target state” of the TME.

6.1 Map each probability vector to the Fisher–Rao information manifold

Mathematically, we transform each distribution using a square-root embedding:

All patients now lie on the unit sphere.

6.2 Compute the Riemannian (Fisher–Rao) mean of responders

This mean represents:

The ideal TSF profile a patient should move towards.

6.3 Compute each patient’s tangent vector

This vector describes:

where they are
how far they are from responders
in which direction they must move
This is “disease state geometry.”

STEP 7 — Foundation Models (CPA): Simulating Drug Effects

We want to know:

If we give a drug, how will each cell type’s expression change?

Foundation models like CPA (Compositional Perturbation Autoencoder) or scGen have been trained on thousands of real perturbation experiments (e.g., L1000).

They can simulate counterfactual expression states:

“What would CD8 T cells look like if exposed to Drug A?”
“What would macrophages look like under IFN-γ stimulation?”
“What happens when NK cells are treated with X?”
We apply the model:

separately to each cell type
using its baseline expression
specifying the drug and dose
This yields new cell-type-specific expression profiles as if the patient had been treated.

STEP 8 — Rebuild the TME Graph Under the Drug

Using the perturbed cell-type expression, we rebuild the LR edges:

Then we:

Extract TSF edges
Compute perturbed TSF probabilities
Map to geometry again
Compute new tangent vector
STEP 9 — Evaluate “Steering Power” of Each Drug

We ask two questions:

9.1 Does the drug move the patient toward the responder manifold?

Measure: distance reduction under Fisher–Rao geometry

9.2 Does the drug move in the correct direction in the tangent space?

Measure: alignment of vectors

A good drug has:

High alignment (correct direction)
Large distance reduction (moves closer)
These define the best drugs for steering that patient’s TME toward a responder-like state.

STEP 10 — Personalized Drug Ranking

For each patient, you produce:

DrugRank table
TSF_before vs TSF_after
Distance_before vs Distance_after
Tangent vectors
Edge-level changes
This becomes a TME steering recommendation.
