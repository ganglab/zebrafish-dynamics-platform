# Zebrafish Dynamics Platform

An interactive static website for visualizing zebrafish whole-brain neural circuit inference and structure-function fusion dynamics results.

The site contains two main analysis modules:

- `Neural Functional Circuit Inference`
  Visualizes inferred directed neural networks in 3D anatomical space from zebrafish calcium imaging data.
- `Structure-Function Fusion: Dynamics Inference`
  Presents governing dynamical equations discovered from inferred network structure using both Symbolic Regression and Network-SINDy.

## Website

Once GitHub Pages is enabled, the site can be accessed at:

- [https://yiting-sun.github.io/zebrafish-dynamics-platform/](https://yiting-sun.github.io/zebrafish-dynamics-platform/)

## Features

- Interactive 3D rendering of inferred neural circuits
- Edge-weight filtering for circuit visualization
- XYZ axis widget and coordinate scale panel
- Dynamics inference comparison across:
  - `gaussian_te`
  - `joint_granger`
  - `dbn_refine`
- Support for two subgraph scales:
  - `N = 200`
  - `N = 500`
- Two dynamics inference methods:
  - `Symbolic Regression`
  - `Network-SINDy`

## Repository Structure

```text
zebrafish-dynamics-platform/
├── index.html
├── README.md
├── .nojekyll
└── data/
    ├── positions.json
    ├── edges_gaussian_te.json
    ├── edges_joint_granger.json
    ├── edges_dbn_refine.json
    ├── network_dynamics_sindy.json
    └── Network_dynamics_sindy.md
```

## Local Preview

Run a local static server from the repository root:

```bash
cd zebrafish-dynamics-platform
python3 -m http.server 8000
```

Then open:

- [http://localhost:8000](http://localhost:8000)

Do not open `index.html` directly with `file://` when testing data-dependent visualization, because local browser security policies may block `fetch()` requests to the JSON files.

## Methods Included in the Site

### Circuit Inference

- `Gaussian TE`
- `Joint Granger`
- `DBN Refine`

### Dynamics Inference

- `Symbolic Regression`
  Equation discovery from graph-informed neuronal activity features
- `Network-SINDy`
  Sparse network dynamics identification with explicit self-dynamics and coupling terms

## Technical Notes

- Frontend is implemented as a single static `index.html`
- 3D rendering uses `Three.js`
- Equation rendering uses `KaTeX`
- Styling uses `Tailwind CSS` via CDN

## License

No license has been added yet. If this repository is intended for public academic reuse, a license should be chosen explicitly.
