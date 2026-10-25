# Principal Component Portfolios of Technology Stocks

A principal-component linear factor model (PCA-LFM) for analyzing common return variation across 14 technology and technology-adjacent equities and constructing factor-mimicking portfolios for the leading principal components.

The project estimates a low-dimensional factor structure from 2024 daily stock returns using exponentially time-decaying flexible probabilities, then evaluates how well the fixed 2024 factor subspace reconstructs realized returns from January through September 2025.

> **Important:** the 2025 exercise is an out-of-sample **reconstruction** test, not a forward return forecast. Factor coordinates are extracted from each day's realized 2025 return vector.

## Project objectives

The notebook asks four main questions:

1. How much of the joint return variation across the equity universe is concentrated in a small number of principal directions?
2. How does volatility-standardizing the returns affect the PCA representation?
3. Does a factor structure estimated from 2024 remain useful for describing returns observed in 2025?
4. Which stocks contribute most strongly to the leading principal-component factors, and how can those factor directions be represented as factor-mimicking portfolios?

## Equity universe

`AAPL`, `AMZN`, `CRM`, `CSCO`, `GOOGL`, `IBM`, `META`, `MSFT`, `NVDA`, `ORCL`, `PLTR`, `SAP`, `SHOP`, and `UBER`.

Adjusted closing prices are downloaded directly with `yfinance`; no separate market-data files are required.

## Methodology

### 1. Return construction

For stock $i$, the daily linear return is,

$$
R_t^{(i)} = \frac{P_t^{(i)}}{P_{t-1}^{(i)}} - 1,
$$

and the multivariate return vector is,

$$
X_t = \begin{pmatrix}R_t^{(1)} & \cdots & R_t^{(14)}\end{pmatrix}^{\top}.
$$

The model is estimated on 2024 returns and evaluated on 2025 observations through September 30, 2025.

### 2. Time-conditioned flexible probabilities

Historical observations receive exponentially decaying weights,

$$
p_t \propto \exp\left(-\frac{\ln 2}{h}a_t\right),
$$

where $a_t$ is the scenario age and $h$ is the half-life. The current implementation uses a half-life equal to one quarter of the 2024 sample length (approximately 63 trading days).

The weighted mean and covariance are:

$$
\widehat\mu_X = \sum_t p_t X_t,
\qquad
\widehat\Sigma_X = \sum_t p_t(X_t-\widehat\mu_X)(X_t-\widehat\mu_X)^{\top}.
$$

The entropy-based effective number of scenarios (ENS) measures the concentration of the flexible probabilities.

### 3. Correlation-based PCA

To prevent higher-volatility stocks from mechanically dominating the decomposition, the covariance matrix is standardized using the diagonal volatility matrix $D$. This makes PCA analyze standardized co-movement rather than absolute volatility magnitude:

$$
C = D^{-1}\widehat\Sigma_XD^{-1}.
$$

The eigendecomposition,

$$
C = E\Lambda E^{\top},
$$

identifies the principal directions of standardized return variation. The project retains the first two components as an intentionally low-dimensional representation.

For the retained eigenvectors $E_k$, the factor-loading and factor-extraction matrices are:

$$
\widehat\beta = D E_k,
\qquad
\widehat\gamma = E_k^{\top}D^{-1}.
$$

Factor realizations and reconstructed returns are then:

$$
\widehat Z_t = \widehat\gamma(X_t-\widehat\mu_X),
\qquad
\widehat X_t = \widehat\mu_X + \widehat\beta\widehat Z_t.
$$

### 4. Out-of-sample reconstruction

The 2024-estimated mean, factor loadings, and extraction matrix are held fixed when applied to 2025 observations. Model stability is evaluated using:

- realized-vs-reconstructed return plots,
- residual time series,
- stock-level reconstruction $R^2$, and
- factor-extraction weights, factor-mimicking portfolio weights, and centered portfolio-return series.

### 5. Factor-mimicking portfolios

The rows of the factor-extraction matrix describe how centered stock returns combine to form each retained principal-component factor. To give these directions a portfolio interpretation, the extraction weights are normalized so that the absolute portfolio weights sum to one:

$$
w_j^{FMP} = \frac{\widehat \gamma_j}{\sum_i | \widehat \gamma_{j,i} |}.
$$

Thus,

$$
\sum_i | w_{j,i}^{FMP} | = 1
$$

The resulting weights preserve the signs and relative magnitudes of the factor-extraction coefficients while expressing them as long and short equity exposures. The centered return of each factor-mimicking portfolio reproduces the corresponding principal-component factor up to a constant scaling factor.


## Selected results

- **251** return scenarios are used in the 2024 estimation sample.
- The time-conditioned probabilities have an **ENS of approximately 191.8**, or **76.4%** of the raw scenario count.
- **PC1 explains approximately 36.92%** of standardized 2024 return variance.
- **PC1 + PC2 explain approximately 46.35%**.
- The first **8 components explain approximately 81.92%**.
- In the 2025 reconstruction exercise, stock-level $R^2$ ranges from approximately **0.20 to 0.72**. AMZN and META are among the best reconstructed stocks, while CRM and UBER retain substantially more residual variation under the two-factor specification.

## Repository structure

```text
principal-component-portfolios/
├── Principal_Component_Portfolios.ipynb
├── README.md
├── requirements.txt
└── .gitignore
```

## Installation

The notebook was developed with Python 3.11. A virtual environment is recommended.

### Windows PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
jupyter lab
```

### macOS / Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
jupyter lab
```

Open `Principal_Component_Portfolios.ipynb` and run the notebook from top to bottom. An internet connection is required for the `yfinance` download.

## Main Python libraries

- NumPy
- pandas
- SciPy
- yfinance
- Matplotlib
- seaborn
- Plotly

## Reproducibility

The PCA estimation and reconstruction calculations are deterministic conditional on the downloaded market data and software environment.

The notebook was developed with Python 3.11 and is designed to run from top to bottom using the dependencies listed in `requirements.txt`.

Historical adjusted-price data are retrieved from `yfinance` at runtime. Because historical market data can be revised by the data provider and library implementations can change over time, later reruns may produce small differences from the saved results. The reported results also depend on the selected equity universe, estimation period, flexible-probability half-life, standardization choice, and retained factor count.

## Interpretation and limitations

- PCA identifies statistical directions of return variation; it does not by itself establish causal economic factors.
- The retained two-factor representation is chosen for interpretation rather than claimed to be an optimal factor count.
- The 2025 analysis evaluates whether a fixed 2024 factor subspace continues to describe later realized returns. It is not a trading backtest or a forecast of returns before they occur.
- The universe is concentrated in technology and technology-adjacent equities, so the leading common component may reflect broader equity-market exposure as well as sector-specific behavior.
- Factor-extraction weights are coefficients used to construct the PCA factors. The project normalizes these coefficients to construct factor-mimicking portfolios, but these portfolios are not intended to represent fully specified tradable strategies. A tradable implementation would require explicit financing, rebalancing, transaction cost, and portfolio-constraint assumptions.

## Possible extensions

- Add volatility-state conditioning and compare joint time/state flexible probabilities with the current time-conditioned baseline.
- Compare correlation-based PCA with covariance-based PCA.
- Select factor count using a formal reconstruction or validation criterion.
- Study rolling factor stability and subspace drift through time.
- Compare principal components with observable market or sector benchmarks.

## Methodological note

The project is informed by Attilio Meucci / ARPM concepts including flexible-probability estimation, linear factor models, and principal portfolios. The implementation and empirical analysis in this repository are the author's own project work.
