# allesfitter — Local Fork

> **This is a personal fork of [allesfitter](https://github.com/MNGuenther/allesfitter) maintained for local use.**  
> It includes modifications to ensure compatibility with modern Python and package versions. It is not intended as a general-purpose distribution.

---

## About this fork

This repository is a locally modified version of *allesfitter* (Günther & Daylan, 2019, ascl:1903.003). Changes from the original include:

- Compatibility fixes for **NumPy 2.x** (`np.VisibleDeprecationWarning` and `np.RankWarning` replacements)
- Installation patches for modern dependency versions (`llvmlite`, `ellc`, `seaborn`, `statsmodels`)

For all official documentation, tutorials, and scientific use, please refer to the **original repository and its resources**:

- 📖 **Documentation:** https://www.allesfitter.com/
- 💻 **Original repository:** https://github.com/MNGuenther/allesfitter
- 📚 **Citations:** see below

---

## Installation

To install this working fork locally, please follow the steps described in **[INSTALLATION.md](INSTALLATION.md)**.

---

## Documentation

All documentation, tutorials, and usage guides are available at the original project website:

🔗 https://www.allesfitter.com/

---

## Citations

If you use *allesfitter* or any part of it in your work, please cite the original authors:

Please cite both the paper and the code, like `\citep{allesfitter-paper, allesfitter-code}`, with:

    @ARTICLE{allesfitter-paper,
     author = {{G{\"u}nther}, Maximilian N. and {Daylan}, Tansu},
     title = "{Allesfitter: Flexible Star and Exoplanet Inference from Photometry and Radial Velocity}",
     journal = {\apjs},
     keywords = {Exoplanets, Binary stars, Stellar flares, Bayesian statistics, Astronomy software, Starspots, Astronomy data modeling, 498, 154, 1603, 1900, 1855, 1572, 1859, Astrophysics - Earth and Planetary Astrophysics, Astrophysics - Instrumentation and Methods for Astrophysics, Astrophysics - Solar and Stellar Astrophysics},
     year = 2021,
     month = may,
     volume = {254},
     number = {1},
     eid = {13},
     pages = {13},
     doi = {10.3847/1538-4365/abe70e},
     archivePrefix = {arXiv},
     eprint = {2003.14371},
     primaryClass = {astro-ph.EP},
     adsurl = {https://ui.adsabs.harvard.edu/abs/2021ApJS..254...13G},
     adsnote = {Provided by the SAO/NASA Astrophysics Data System}
    }

    @MISC{allesfitter-code,
     author = {{G{\"u}nther}, Maximilian~N. and {Daylan}, Tansu},
     title = "{Allesfitter: Flexible Star and Exoplanet Inference From Photometry and Radial Velocity}",
     keywords = {Software },
     howpublished = {Astrophysics Source Code Library},
     year = 2019,
     month = mar,
     archivePrefix = "ascl",
     eprint = {1903.003},
     adsurl = {http://adsabs.harvard.edu/abs/2019ascl.soft03003G},
     adsnote = {Provided by the SAO/NASA Astrophysics Data System}
    }

**Additional software acknowledgements**:

    - ellc: Maxted, P. F. L. (2016), Astronomy and Astrophysics, 591, A111
    - aflare: Davenport, J. R. A. et al. (2014), The Astrophysical Journal, 797, 122
    - dynesty: Speagel, J. (2019), arXiv:1904.02180
    - emcee: Foreman-Mackey, D., et al. (2013), Publications of the Astronomical Society of the Pacific, 125, 306
    - celerite: Foreman-Mackey, D., et al. (2017), The Astronomical Journal, 154, 220
    - corner: Foreman-Mackey, D., et al.
    - python: Rossum G. (1995), Technical Report, Python Reference Manual, Amsterdam, The Netherlands
    - numpy: van der Walt S., et al. (2011), Comput. Sci. Eng., 13, 22
    - scipy: Jones E. et al. (2001), SciPy: Open Source Scientific tools for Python. Available at: http://www.scipy.org/
    - matplotlib: Hunter J. D. (2007), Comput. Sci. Eng., 9, 90
    - tqdm: doi:10.5281/zenodo.1468033
    - seaborn: https://seaborn.pydata.org/index.html

---

## Original authors

Maximilian N. Günther & Tansu Daylan

**License:** MIT — see https://github.com/MNGuenther/allesfitter
