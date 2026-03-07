# BT56 — Meta-Research Engine: Automated Systematic Review & Reproducibility Auditor for Educational Neuroscience

**Domain:** Meta-Research / Research Methodology / NLP / Reproducibility Science  
**Date:** 2026-02-27  
**Status:** Brainstorming  
**Novelty Level:** ★★★★★ (Very High)  
**Feasibility:** ★★★★☆ (Achievable with existing LLM + NLP tools)

---

## PART A — WHAT & WHY

### A1. The Problem

Educational neuroscience is a young, interdisciplinary field with a severe reproducibility crisis. Studies often: (1) use small samples (median n=23), (2) apply inappropriate statistical tests, (3) make causal claims from correlational data, (4) fail to correct for multiple comparisons, and (5) selectively report only significant results. Manual systematic reviews take 6-18 months and cannot keep pace with the 10,000+ papers published annually in this domain.

**The gap:** No automated tool can ingest a corpus of educational neuroscience papers and produce a comprehensive reproducibility audit — quantifying statistical rigor, detecting p-hacking signals, assessing sample size adequacy, identifying logical fallacies in causal reasoning, and flagging citation networks that form "echo chambers."

### A2. Why It Matters

| Stakeholder | Pain Point |
|---|---|
| Researchers | Cannot assess whether foundational findings are reliable |
| Journal editors | No automated screening for statistical red flags |
| Grant agencies | Fund research that may not replicate |
| Policymakers | Base education policy on unreliable evidence |
| Students/Early-career | Cannot identify which methods/claims to trust |

### A3. Research Gap

| Existing Work | Limitation |
|---|---|
| Cochrane systematic reviews | Manual, takes 6-18 months |
| statcheck (R package) | Only checks p-value consistency, not methodology |
| GRIM test | Only checks mean/SD plausibility for integers |
| scite.ai | Classifies citations as supporting/contrasting, no methodology audit |
| ASReview | Active learning for screening, not methodology assessment |
| Semantic Scholar | Discovery tool, no reproducibility analysis |

**Our innovation:** An end-to-end automated meta-research pipeline that (1) extracts structured methodology data from PDFs using LLM-assisted parsing, (2) applies 15 reproducibility checks, (3) detects p-hacking via p-curve analysis, (4) identifies causal language misuse, (5) maps citation echo chambers, and (6) generates a quantitative Reproducibility Confidence Score (RCS) for each paper and the field as a whole.

### A4. Core Hypothesis

> *An automated meta-research engine can assess reproducibility indicators in educational neuroscience papers with > 85% agreement with expert human reviewers, process 1000 papers in < 24 hours, and identify systematic field-level biases invisible to individual reviewers.*

---

## PART B — TECHNICAL APPROACH

### B1. Mathematical Framework

#### P-Curve Analysis

**Expected p-curve under true effect (right-skewed):**

$$f(p | d > 0) = 1 + \frac{d \cdot \Phi^{-1}(1 - p/2)}{p} \cdot \phi(\Phi^{-1}(1 - p/2) - d)$$

Where $d$ is Cohen's d, $\Phi^{-1}$ is the probit function, $\phi$ is the standard normal PDF.

**Expected p-curve under null (uniform):**

$$f(p | d = 0) = 1 \quad \forall p \in (0, 0.05]$$

**P-curve evidential value test:**

$$\chi^2_{right} = -2 \sum_{i=1}^{k} \ln(p_i), \quad df = 2k$$

If $\chi^2_{right}$ exceeds critical value, the set of findings has evidential value.

#### Reproducibility Confidence Score (RCS)

$$RCS = \sum_{j=1}^{15} w_j \cdot C_j$$

Where $C_j \in [0, 1]$ is the score for check $j$, $w_j$ is the importance weight, $\sum w_j = 1$.

| Check $j$ | Name | Weight $w_j$ |
|---|---|---|
| 1 | Sample size adequacy | 0.12 |
| 2 | Statistical test appropriateness | 0.10 |
| 3 | Multiple comparison correction | 0.08 |
| 4 | Effect size reporting | 0.08 |
| 5 | Confidence intervals reported | 0.06 |
| 6 | P-value consistency (statcheck) | 0.08 |
| 7 | Causal language appropriate | 0.10 |
| 8 | Pre-registration status | 0.07 |
| 9 | Data/code availability | 0.06 |
| 10 | Blinding/randomization | 0.05 |
| 11 | Attrition/dropout reported | 0.04 |
| 12 | Replication statement | 0.04 |
| 13 | GRIM/SPRITE consistency | 0.04 |
| 14 | Conflict of interest declaration | 0.04 |
| 15 | Citation diversity | 0.04 |

#### Citation Echo Chamber Detection

**Citation network modularity:**

$$Q = \frac{1}{2m} \sum_{ij} \left[ A_{ij} - \frac{k_i k_j}{2m} \right] \delta(c_i, c_j)$$

High modularity ($Q > 0.6$) with small clusters suggests echo chambers.

**Self-citation ratio:**

$$SCR = \frac{N_{self-cite}}{N_{total-cite}}$$

Field norm: $SCR \approx 0.1$; concerning if $SCR > 0.25$.

### B2. Pipeline Architecture

```
                     PDF CORPUS
                         │
                         ▼
┌─────────────────────────────────────────────────┐
│  STAGE 1: PDF PARSING & STRUCTURED EXTRACTION    │
│  ┌─────────┐  ┌─────────────┐  ┌────────────┐  │
│  │ GROBID   │  │ LLM-assisted│  │ Table/Fig  │  │
│  │ header   │  │ methodology │  │ extractor  │  │
│  │ parsing  │  │ extraction  │  │            │  │
│  └────┬────┘  └──────┬──────┘  └─────┬──────┘  │
│       └──────────────┼────────────────┘          │
│                      ▼                            │
│          Structured Paper Object                  │
│  {sample_size, design, statistics, p_values,     │
│   effect_sizes, conclusions, references}         │
└──────────────────────┬──────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────┐
│  STAGE 2: 15-POINT REPRODUCIBILITY AUDIT         │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ │
│  │Sample│ │Stats │ │Causal│ │p-hack│ │Data  │ │
│  │Size  │ │Valid │ │Logic │ │Detect│ │Avail │ │
│  └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘ │
│     └────────┴────────┴────────┴────────┘      │
│                      │                           │
│              Reproducibility                     │
│            Confidence Score (RCS)                │
└──────────────────────┬──────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────┐
│  STAGE 3: FIELD-LEVEL META-ANALYSIS              │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────┐│
│  │ P-curve   │ │ Citation │ │ Decline effect   ││
│  │ analysis  │ │ network  │ │ (time-series)    ││
│  └─────┬────┘ └────┬─────┘ └────────┬─────────┘│
│        └───────────┼─────────────────┘           │
│                    ▼                              │
│     Field Health Report + Recommendations        │
└─────────────────────────────────────────────────┘
```

### B3. Python Implementation

```python
"""
BT56 - Meta-Research Engine: Automated Reproducibility Auditor
for Educational Neuroscience Literature
"""

import numpy as np
from dataclasses import dataclass, field
from typing import Dict, List, Tuple, Optional, Set
from scipy import stats as scipy_stats
from collections import Counter
import re
import warnings
warnings.filterwarnings('ignore')


@dataclass
class ExtractedPaper:
    """Structured data extracted from a research paper."""
    paper_id: str
    title: str
    year: int
    authors: List[str]
    journal: str
    
    # Methodology
    study_design: str = "observational"  # experimental, quasi, observational
    sample_size: int = 0
    groups: int = 2
    measures: int = 1
    
    # Statistics
    p_values: List[float] = field(default_factory=list)
    effect_sizes: List[float] = field(default_factory=list)
    confidence_intervals: List[Tuple[float, float]] = field(default_factory=list)
    test_types: List[str] = field(default_factory=list)
    
    # Quality indicators
    pre_registered: bool = False
    data_available: bool = False
    code_available: bool = False
    blinded: bool = False
    randomized: bool = False
    multiple_comparison_corrected: bool = False
    attrition_reported: bool = False
    conflict_declared: bool = False
    
    # Causal language
    causal_claims: List[str] = field(default_factory=list)
    
    # References
    references: List[str] = field(default_factory=list)
    self_citations: int = 0


class SampleSizeChecker:
    """Check if sample size is adequate for claimed analyses."""
    
    @staticmethod
    def required_n_ttest(d: float = 0.5, alpha: float = 0.05, 
                          power: float = 0.80) -> int:
        """Minimum n per group for two-sample t-test."""
        z_alpha = scipy_stats.norm.ppf(1 - alpha / 2)
        z_beta = scipy_stats.norm.ppf(power)
        n = 2 * ((z_alpha + z_beta) / d) ** 2
        return int(np.ceil(n))
    
    @staticmethod
    def required_n_correlation(r: float = 0.3, alpha: float = 0.05,
                                power: float = 0.80) -> int:
        """Minimum n for correlation."""
        z_alpha = scipy_stats.norm.ppf(1 - alpha / 2)
        z_beta = scipy_stats.norm.ppf(power)
        z_r = 0.5 * np.log((1 + r) / (1 - r))  # Fisher z
        n = ((z_alpha + z_beta) / z_r) ** 2 + 3
        return int(np.ceil(n))
    
    def check(self, paper: ExtractedPaper) -> Dict:
        """Assess sample size adequacy."""
        if not paper.effect_sizes:
            median_d = 0.5  # Assume medium effect
        else:
            median_d = np.median(paper.effect_sizes)
        
        required_n = self.required_n_ttest(max(median_d, 0.1))
        ratio = paper.sample_size / max(required_n, 1)
        
        score = min(1.0, ratio)
        
        return {
            'score': score,
            'sample_size': paper.sample_size,
            'required_n': required_n,
            'ratio': ratio,
            'adequate': ratio >= 1.0,
            'comment': f"n={paper.sample_size}, need {required_n} for d={median_d:.2f}"
        }


class StatisticalConsistencyChecker:
    """Check statistical reporting consistency (statcheck-like)."""
    
    @staticmethod
    def check_p_from_t(t_stat: float, df: int, reported_p: float,
                        tolerance: float = 0.05) -> bool:
        """Verify if reported p-value matches t-statistic and df."""
        computed_p = 2 * scipy_stats.t.sf(abs(t_stat), df)
        return abs(computed_p - reported_p) < tolerance
    
    @staticmethod
    def check_p_from_f(f_stat: float, df1: int, df2: int, 
                        reported_p: float, tolerance: float = 0.05) -> bool:
        """Verify if reported p-value matches F-statistic."""
        computed_p = scipy_stats.f.sf(f_stat, df1, df2)
        return abs(computed_p - reported_p) < tolerance
    
    def check_grim(self, mean: float, n: int, n_decimals: int = 2) -> bool:
        """GRIM test: check if mean of integers is possible."""
        # Mean of n integers must be a multiple of 1/n
        granularity = 1 / n
        remainder = (mean / granularity) % 1
        return remainder < 10 ** (-n_decimals) or remainder > 1 - 10 ** (-n_decimals)
    
    def check_paper(self, paper: ExtractedPaper) -> Dict:
        """Run consistency checks on a paper."""
        n_checks = len(paper.p_values)
        if n_checks == 0:
            return {'score': 0.5, 'n_checks': 0, 'comment': 'No p-values to check'}
        
        # Simulate consistency check (in real system, would parse actual stats)
        inconsistencies = 0
        for p in paper.p_values:
            # Simplified: flag rounded p-values (suspicious)
            p_str = f"{p:.4f}"
            if p_str.endswith('000') or p_str.endswith('500'):
                inconsistencies += 1
        
        score = 1.0 - (inconsistencies / max(n_checks, 1))
        
        return {
            'score': max(0, score),
            'n_checks': n_checks,
            'inconsistencies': inconsistencies,
            'comment': f"{inconsistencies}/{n_checks} suspicious p-values"
        }


class CausalLanguageDetector:
    """Detect inappropriate causal language in correlational studies."""
    
    CAUSAL_INDICATORS = [
        'causes', 'caused', 'leads to', 'results in', 'produces',
        'increases', 'decreases', 'improves', 'reduces', 'affects',
        'influences', 'determines', 'drives', 'triggers', 'induces',
        'effect of', 'impact of', 'consequence of'
    ]
    
    HEDGING_WORDS = [
        'may', 'might', 'could', 'suggests', 'associated with',
        'correlated with', 'related to', 'linked to', 'appears to'
    ]
    
    def detect(self, paper: ExtractedPaper) -> Dict:
        """Check for causal language misuse."""
        is_experimental = paper.study_design == "experimental" and paper.randomized
        
        causal_count = len(paper.causal_claims)
        
        if is_experimental:
            # Causal language is appropriate for RCTs
            score = 1.0
            comment = "Experimental design — causal language appropriate"
        else:
            if causal_count == 0:
                score = 1.0
                comment = "No causal claims detected"
            else:
                # Penalize causal language in non-experimental designs
                severity = min(1.0, causal_count / 5)
                score = 1.0 - severity
                comment = f"{causal_count} causal claims in {paper.study_design} study"
        
        return {
            'score': score,
            'causal_claims': causal_count,
            'design': paper.study_design,
            'appropriate': is_experimental or causal_count == 0,
            'comment': comment
        }


class PHackingDetector:
    """Detect p-hacking signals in a collection of p-values."""
    
    def p_curve_test(self, p_values: np.ndarray) -> Dict:
        """Run p-curve analysis on significant p-values."""
        # Filter to significant results
        sig_p = p_values[p_values < 0.05]
        
        if len(sig_p) < 5:
            return {
                'test': 'p-curve',
                'n_significant': len(sig_p),
                'evidential_value': None,
                'comment': 'Too few significant p-values for p-curve'
            }
        
        # Right-skew test (continuous): pp-values
        pp_values = sig_p / 0.05  # Normalize to [0, 1]
        
        # Under true effect: pp-values should be right-skewed (concentrated near 0)
        # Under null: pp-values should be uniform
        
        # Binomial test: proportion below 0.025
        n_below = np.sum(sig_p < 0.025)
        binom_p = scipy_stats.binom_test(n_below, len(sig_p), 0.5)
        
        # KS test against uniform
        ks_stat, ks_p = scipy_stats.kstest(pp_values, 'uniform')
        
        # Fisher's method
        chi2_stat = -2 * np.sum(np.log(pp_values + 1e-10))
        df = 2 * len(pp_values)
        fisher_p = scipy_stats.chi2.sf(chi2_stat, df)
        
        has_evidential_value = fisher_p < 0.05
        
        return {
            'test': 'p-curve',
            'n_significant': len(sig_p),
            'prop_below_025': n_below / len(sig_p),
            'binomial_p': binom_p,
            'ks_stat': ks_stat,
            'ks_p': ks_p,
            'fisher_chi2': chi2_stat,
            'fisher_p': fisher_p,
            'evidential_value': has_evidential_value,
            'comment': ('Right-skewed (true effect)' if has_evidential_value 
                       else 'Flat/left-skewed (possible p-hacking)')
        }
    
    def caliper_test(self, p_values: np.ndarray, 
                      window: float = 0.005) -> Dict:
        """Detect excess of p-values just below 0.05 (caliper test)."""
        n_just_below = np.sum((p_values > 0.04) & (p_values <= 0.05))
        n_just_above = np.sum((p_values > 0.05) & (p_values <= 0.06))
        
        # Under no p-hacking: ratio should be ~1
        if n_just_above == 0:
            ratio = float('inf') if n_just_below > 0 else 1.0
        else:
            ratio = n_just_below / n_just_above
        
        suspicious = ratio > 2.0
        
        return {
            'test': 'caliper',
            'n_just_below_05': n_just_below,
            'n_just_above_05': n_just_above,
            'ratio': ratio,
            'suspicious': suspicious,
            'comment': f'Ratio: {ratio:.2f}' + (' — suspicious clustering below .05' if suspicious else '')
        }


class CitationNetworkAnalyzer:
    """Analyze citation patterns for echo chambers and diversity."""
    
    def __init__(self):
        self.adjacency = {}  # paper_id -> set of cited paper_ids
    
    def add_paper(self, paper: ExtractedPaper):
        """Add paper to citation network."""
        self.adjacency[paper.paper_id] = set(paper.references)
    
    def self_citation_ratio(self, paper: ExtractedPaper) -> float:
        """Calculate self-citation ratio."""
        if not paper.references:
            return 0.0
        return paper.self_citations / len(paper.references)
    
    def citation_diversity(self, paper: ExtractedPaper) -> Dict:
        """Assess citation diversity."""
        n_refs = len(paper.references)
        
        if n_refs == 0:
            return {'score': 0, 'n_refs': 0}
        
        scr = self.self_citation_ratio(paper)
        
        # Simulate: unique author groups cited
        unique_groups = int(n_refs * 0.6 + np.random.randint(0, n_refs // 3 + 1))
        diversity_index = unique_groups / max(n_refs, 1)
        
        score = 0.5 * (1 - min(scr / 0.3, 1.0)) + 0.5 * diversity_index
        
        return {
            'score': score,
            'n_refs': n_refs,
            'self_citation_ratio': scr,
            'diversity_index': diversity_index,
            'echo_chamber_risk': scr > 0.25 or diversity_index < 0.3
        }


class ReproducibilityEngine:
    """Complete meta-research reproducibility auditor."""
    
    def __init__(self):
        self.sample_checker = SampleSizeChecker()
        self.stat_checker = StatisticalConsistencyChecker()
        self.causal_detector = CausalLanguageDetector()
        self.phacking_detector = PHackingDetector()
        self.citation_analyzer = CitationNetworkAnalyzer()
        
        # Weights for RCS
        self.weights = {
            'sample_size': 0.12,
            'stat_validity': 0.10,
            'multiple_comparison': 0.08,
            'effect_size_reporting': 0.08,
            'ci_reporting': 0.06,
            'p_consistency': 0.08,
            'causal_language': 0.10,
            'pre_registration': 0.07,
            'data_availability': 0.06,
            'blinding': 0.05,
            'attrition': 0.04,
            'replication': 0.04,
            'grim_sprite': 0.04,
            'conflict_interest': 0.04,
            'citation_diversity': 0.04
        }
    
    def audit_paper(self, paper: ExtractedPaper) -> Dict:
        """Generate full reproducibility audit for one paper."""
        checks = {}
        
        # 1. Sample size adequacy
        checks['sample_size'] = self.sample_checker.check(paper)
        
        # 2. Statistical consistency
        checks['p_consistency'] = self.stat_checker.check_paper(paper)
        
        # 3. Causal language
        checks['causal_language'] = self.causal_detector.detect(paper)
        
        # 4. Multiple comparison correction
        checks['multiple_comparison'] = {
            'score': 1.0 if paper.multiple_comparison_corrected or len(paper.p_values) <= 1 
                     else 0.3,
            'corrected': paper.multiple_comparison_corrected,
            'n_tests': len(paper.p_values)
        }
        
        # 5. Effect size reporting
        checks['effect_size_reporting'] = {
            'score': 1.0 if paper.effect_sizes else 0.0,
            'reported': bool(paper.effect_sizes),
            'n_effect_sizes': len(paper.effect_sizes)
        }
        
        # 6. CI reporting
        checks['ci_reporting'] = {
            'score': 1.0 if paper.confidence_intervals else 0.0,
            'reported': bool(paper.confidence_intervals)
        }
        
        # 7. Pre-registration
        checks['pre_registration'] = {
            'score': 1.0 if paper.pre_registered else 0.0,
        }
        
        # 8. Data/code availability
        data_score = (0.5 * paper.data_available + 0.5 * paper.code_available)
        checks['data_availability'] = {
            'score': data_score,
            'data': paper.data_available,
            'code': paper.code_available
        }
        
        # 9. Blinding
        checks['blinding'] = {
            'score': 1.0 if paper.blinded or paper.study_design == "observational" else 0.3
        }
        
        # 10. Attrition
        checks['attrition'] = {
            'score': 1.0 if paper.attrition_reported else 0.3
        }
        
        # 11. GRIM/SPRITE (simplified)
        checks['grim_sprite'] = {
            'score': 0.8 + 0.2 * np.random.random()  # Simulated
        }
        
        # 12. Replication statement
        checks['replication'] = {
            'score': 0.5  # Neutral if not stated
        }
        
        # 13. Conflict of interest
        checks['conflict_interest'] = {
            'score': 1.0 if paper.conflict_declared else 0.5
        }
        
        # 14. Citation diversity
        checks['citation_diversity'] = self.citation_analyzer.citation_diversity(paper)
        
        # Calculate RCS
        rcs = 0
        for check_name, weight in self.weights.items():
            if check_name in checks:
                rcs += weight * checks[check_name].get('score', 0.5)
            else:
                rcs += weight * 0.5
        
        # RCS interpretation
        if rcs >= 0.8:
            rating = "★★★★★ Excellent"
        elif rcs >= 0.65:
            rating = "★★★★☆ Good"
        elif rcs >= 0.50:
            rating = "★★★☆☆ Fair"
        elif rcs >= 0.35:
            rating = "★★☆☆☆ Concerning"
        else:
            rating = "★☆☆☆☆ Poor"
        
        return {
            'paper_id': paper.paper_id,
            'title': paper.title,
            'rcs': rcs,
            'rating': rating,
            'checks': checks
        }
    
    def field_analysis(self, papers: List[ExtractedPaper]) -> Dict:
        """Analyze field-level reproducibility patterns."""
        all_p_values = []
        rcs_scores = []
        sample_sizes = []
        yearly_rcs = {}
        
        for paper in papers:
            all_p_values.extend(paper.p_values)
            sample_sizes.append(paper.sample_size)
            
            audit = self.audit_paper(paper)
            rcs_scores.append(audit['rcs'])
            
            if paper.year not in yearly_rcs:
                yearly_rcs[paper.year] = []
            yearly_rcs[paper.year].append(audit['rcs'])
        
        # P-curve analysis
        all_p = np.array(all_p_values)
        p_curve = self.phacking_detector.p_curve_test(all_p)
        caliper = self.phacking_detector.caliper_test(all_p)
        
        # Temporal trends
        years_sorted = sorted(yearly_rcs.keys())
        yearly_means = [np.mean(yearly_rcs[y]) for y in years_sorted]
        
        # Decline effect check
        if len(rcs_scores) > 10:
            slope, _, r_value, p_value, _ = scipy_stats.linregress(
                range(len(rcs_scores)), rcs_scores)
            decline_effect = slope < 0 and p_value < 0.05
        else:
            slope, r_value, p_value = 0, 0, 1
            decline_effect = False
        
        return {
            'n_papers': len(papers),
            'median_sample_size': np.median(sample_sizes),
            'mean_rcs': np.mean(rcs_scores),
            'std_rcs': np.std(rcs_scores),
            'rcs_below_fair': np.mean(np.array(rcs_scores) < 0.5) * 100,
            'p_curve': p_curve,
            'caliper_test': caliper,
            'yearly_rcs': dict(zip(years_sorted, yearly_means)),
            'decline_effect': {
                'detected': decline_effect,
                'slope': slope,
                'r': r_value,
                'p': p_value
            },
            'pre_registration_rate': np.mean([p.pre_registered for p in papers]) * 100,
            'data_sharing_rate': np.mean([p.data_available for p in papers]) * 100,
            'effect_size_reporting_rate': np.mean([bool(p.effect_sizes) for p in papers]) * 100
        }


def generate_synthetic_papers(n: int = 100) -> List[ExtractedPaper]:
    """Generate synthetic educational neuroscience papers for testing."""
    papers = []
    
    designs = ['experimental', 'quasi-experimental', 'observational', 'correlational']
    journals = ['NeuroEducation', 'Mind Brain Ed', 'J Edu Psych', 
                'Trends Neurosci Ed', 'Frontiers Ed']
    
    for i in range(n):
        year = np.random.randint(2010, 2026)
        design = np.random.choice(designs, p=[0.15, 0.25, 0.35, 0.25])
        
        # Sample size (often small in this field)
        sample_size = int(np.random.lognormal(3.2, 0.8))  # Median ~25
        sample_size = max(10, min(sample_size, 500))
        
        # P-values (with some p-hacking simulation)
        n_tests = np.random.randint(3, 15)
        p_vals = []
        for _ in range(n_tests):
            if np.random.random() < 0.3:  # 30% chance of p-hacking
                p = np.random.uniform(0.01, 0.049)  # Just significant
            else:
                p = np.random.exponential(0.1)
            p_vals.append(min(p, 1.0))
        
        # Effect sizes (not always reported)
        if np.random.random() < 0.6:
            effect_sizes = [abs(np.random.normal(0.4, 0.3)) for _ in range(n_tests // 2)]
        else:
            effect_sizes = []
        
        n_refs = np.random.randint(15, 60)
        
        # Quality indicators (improve with year)
        year_factor = (year - 2010) / 15
        
        paper = ExtractedPaper(
            paper_id=f"PAPER-{i:04d}",
            title=f"Synthetic Paper {i}: {design} study",
            year=year,
            authors=[f"Author_{j}" for j in range(np.random.randint(2, 6))],
            journal=np.random.choice(journals),
            study_design=design,
            sample_size=sample_size,
            groups=np.random.randint(2, 5),
            measures=np.random.randint(1, 8),
            p_values=p_vals,
            effect_sizes=effect_sizes,
            confidence_intervals=[(0, 1)] * len(effect_sizes) if np.random.random() < 0.4 else [],
            test_types=['t-test', 'ANOVA', 'correlation'][:n_tests],
            pre_registered=np.random.random() < (0.05 + 0.3 * year_factor),
            data_available=np.random.random() < (0.1 + 0.2 * year_factor),
            code_available=np.random.random() < (0.05 + 0.15 * year_factor),
            blinded=np.random.random() < 0.3 if design == 'experimental' else False,
            randomized=np.random.random() < 0.7 if design == 'experimental' else False,
            multiple_comparison_corrected=np.random.random() < (0.3 + 0.3 * year_factor),
            attrition_reported=np.random.random() < 0.5,
            conflict_declared=np.random.random() < 0.7,
            causal_claims=[f"claim_{j}" for j in range(
                np.random.randint(0, 3 if design in ['experimental'] else 5))],
            references=[f"REF-{np.random.randint(0, 1000):04d}" for _ in range(n_refs)],
            self_citations=np.random.randint(0, max(1, n_refs // 5))
        )
        
        papers.append(paper)
    
    return papers


def run_full_simulation():
    """Execute complete meta-research engine simulation."""
    print("=" * 70)
    print("BT56: META-RESEARCH ENGINE — REPRODUCIBILITY AUDITOR")
    print("       Educational Neuroscience Literature Analysis")
    print("=" * 70)
    
    # Generate synthetic corpus
    papers = generate_synthetic_papers(n=200)
    print(f"\nGenerated {len(papers)} synthetic educational neuroscience papers")
    
    # Initialize engine
    engine = ReproducibilityEngine()
    
    # Audit all papers
    audits = [engine.audit_paper(p) for p in papers]
    
    # Individual paper examples
    print(f"\n╔══════════════════════════════════════════════════════════╗")
    print(f"║     SAMPLE INDIVIDUAL PAPER AUDITS                       ║")
    print(f"╠══════════════════════════════════════════════════════════╣")
    
    for audit in audits[:5]:
        print(f"║ {audit['paper_id']}: RCS = {audit['rcs']:.3f} {audit['rating']}    ║")
        checks = audit['checks']
        print(f"║   Sample: {checks['sample_size'].get('score', 0):.2f} | "
              f"Stats: {checks['p_consistency'].get('score', 0):.2f} | "
              f"Causal: {checks['causal_language'].get('score', 0):.2f} | "
              f"MultiComp: {checks['multiple_comparison'].get('score', 0):.2f}  ║")
        print(f"║{'─' * 58}║")
    
    print(f"╚══════════════════════════════════════════════════════════╝")
    
    # Field-level analysis
    field = engine.field_analysis(papers)
    
    print(f"\n╔══════════════════════════════════════════════════════════╗")
    print(f"║        FIELD-LEVEL ANALYSIS (N={field['n_papers']})                  ║")
    print(f"╠══════════════════════════════════════════════════════════╣")
    print(f"║ Median sample size:       {field['median_sample_size']:>6.0f}                      ║")
    print(f"║ Mean RCS:                 {field['mean_rcs']:>6.3f} ± {field['std_rcs']:.3f}              ║")
    print(f"║ Papers below 'Fair':      {field['rcs_below_fair']:>5.1f}%                       ║")
    print(f"║ Pre-registration rate:    {field['pre_registration_rate']:>5.1f}%                       ║")
    print(f"║ Data sharing rate:        {field['data_sharing_rate']:>5.1f}%                       ║")
    print(f"║ Effect size reporting:    {field['effect_size_reporting_rate']:>5.1f}%                       ║")
    print(f"╠══════════════════════════════════════════════════════════╣")
    
    # P-curve results
    pc = field['p_curve']
    print(f"║ P-CURVE ANALYSIS                                         ║")
    print(f"║   Significant p-values:   {pc['n_significant']:>6d}                      ║")
    if pc['evidential_value'] is not None:
        print(f"║   Evidential value:       {'✓ YES' if pc['evidential_value'] else '✗ NO':>6s}"
              f"                      ║")
        print(f"║   Fisher χ²:             {pc['fisher_chi2']:>8.1f}                    ║")
        print(f"║   p-value (right skew):  {pc['fisher_p']:>8.4f}                    ║")
        print(f"║   % below .025:          {pc['prop_below_025'] * 100:>5.1f}%                       ║")
    
    # Caliper test
    ct = field['caliper_test']
    print(f"║ CALIPER TEST                                             ║")
    print(f"║   Just below .05:         {ct['n_just_below_05']:>6d}                      ║")
    print(f"║   Just above .05:         {ct['n_just_above_05']:>6d}                      ║")
    print(f"║   Ratio:                  {ct['ratio']:>6.2f}"
          f"{'  ⚠ SUSPICIOUS' if ct['suspicious'] else '  ✓ Normal':>20s}    ║")
    
    # Decline effect
    de = field['decline_effect']
    print(f"║ DECLINE EFFECT                                           ║")
    print(f"║   Detected:               {'✓ YES' if de['detected'] else '✗ NO':>6s}"
          f"                      ║")
    print(f"║   Slope:                  {de['slope']:>8.4f}                    ║")
    print(f"╚══════════════════════════════════════════════════════════╝")
    
    # RCS distribution
    rcs_all = [a['rcs'] for a in audits]
    print(f"\n--- RCS Distribution ---\n")
    bins = [(0, 0.35, "Poor"), (0.35, 0.50, "Concerning"), 
            (0.50, 0.65, "Fair"), (0.65, 0.80, "Good"), (0.80, 1.0, "Excellent")]
    
    for low, high, label in bins:
        count = sum(1 for r in rcs_all if low <= r < high)
        bar = "█" * (count // 2)
        print(f"  {label:<12s} [{low:.2f}-{high:.2f}]: {count:>4d} ({count/len(rcs_all)*100:>4.1f}%) {bar}")
    
    # Yearly trends
    print(f"\n--- Yearly RCS Trends ---\n")
    for year, mean_rcs in sorted(field['yearly_rcs'].items()):
        bar = "█" * int(mean_rcs * 30)
        print(f"  {year}: {mean_rcs:.3f} {bar}")
    
    return field, audits


if __name__ == '__main__':
    field_results, paper_audits = run_full_simulation()
```

---

## PART C — EXPECTED RESULTS

### C1. Engine Performance

| Metric | Target | Expected |
|---|---|---|
| Agreement with human reviewers | > 85% | 87-92% |
| Paper processing time | < 30 sec/paper | 10-20 sec/paper |
| Throughput (24h) | > 1000 papers | 2000-5000 papers |
| False positive rate (flags) | < 10% | 5-8% |
| P-value consistency detection | > 90% | 93% (vs. statcheck) |
| Causal language detection | > 80% | 83-88% |

### C2. Field-Level Findings (Expected from Ed-Neuro Corpus)

| Indicator | Expected Finding |
|---|---|
| Median sample size | 20-35 (underpowered) |
| Mean RCS | 0.45-0.55 (Fair) |
| Pre-registration rate | 5-15% (improving) |
| Data sharing rate | 10-20% |
| P-hacking signal (caliper ratio) | 1.5-2.5× (moderate) |
| Causal language misuse | 30-40% of correlational studies |
| Echo chamber clusters | 3-5 identified |

### C3. Temporal Trends

| Period | Mean RCS | Pre-Reg Rate | Data Sharing |
|---|---|---|---|
| 2010-2015 | 0.40-0.45 | < 3% | < 5% |
| 2015-2020 | 0.48-0.55 | 5-10% | 10-15% |
| 2020-2026 | 0.55-0.65 | 15-25% | 20-30% |

---

## PART D — COMPARISON WITH EXISTING WORK

| Feature | statcheck | GRIM | scite.ai | ASReview | Cochrane | **BT56 (Ours)** |
|---|---|---|---|---|---|---|
| P-value consistency | ✓ | ✗ | ✗ | ✗ | Manual | **✓** |
| Sample size adequacy | ✗ | ✗ | ✗ | ✗ | ✓ | **✓** |
| P-hacking detection | ✗ | ✗ | ✗ | ✗ | ✗ | **✓ (p-curve + caliper)** |
| Causal language audit | ✗ | ✗ | ✗ | ✗ | Manual | **✓ (NLP)** |
| Citation network | ✗ | ✗ | Partial | ✗ | ✗ | **✓ (echo chamber)** |
| Composite score (RCS) | ✗ | ✗ | ✗ | ✗ | ✗ | **✓ (15-point)** |
| Field-level analysis | ✗ | ✗ | ✗ | ✗ | ✓ | **✓ (automated)** |
| Automated (no human) | ✓ | ✓ | ✓ | Partial | ✗ | **✓** |
| Speed (papers/hour) | 1000+ | 1000+ | N/A | ~100 | ~0.1 | **200-500** |

---

## PART E — TOOLS & RESOURCES

### E1. Software Stack

| Tool | Purpose |
|---|---|
| Python + NumPy/SciPy | Statistical analysis core |
| GROBID | PDF → structured XML extraction |
| GPT-4 / Claude API | Methodology section parsing |
| spaCy / scispaCy | NLP for causal language detection |
| NetworkX + Louvain | Citation network analysis |
| Semantic Scholar API | Paper metadata & citation graph |
| CrossRef API | DOI resolution & reference linking |
| PostgreSQL | Paper database |
| FastAPI | REST API for auditing service |
| Streamlit | Interactive dashboard |

### E2. Data Sources

| Source | Coverage |
|---|---|
| PubMed Central (PMC) | Open-access biomedical papers |
| Semantic Scholar | 200M+ papers metadata |
| OpenAlex | Open bibliometric database |
| ERIC | Education research database |
| Unpaywall | Open-access full-text links |

### E3. Publication Targets

| Venue | Type | Fit |
|---|---|---|
| Nature Human Behaviour | Journal | ★★★★★ |
| Meta-Psychology | Journal | ★★★★★ |
| Royal Society Open Science | Journal | ★★★★☆ |
| Scientometrics | Journal | ★★★★☆ |
| EMNLP (NLP conference) | Conference | ★★★☆☆ |

### E4. Summary Metrics

| Dimension | Rating |
|---|---|
| Effort | 🟡 Medium (NLP pipeline + statistical engine) |
| Difficulty | 🟡 Medium (LLM integration + validation) |
| Novelty | 🔴 Very High (comprehensive automated audit) |
| Impact | 🔴 Very High (field-transforming for reproducibility) |
| Time to Prototype | 2-3 months |
| Time to Publication | 4-6 months |
