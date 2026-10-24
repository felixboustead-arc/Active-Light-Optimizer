# Third-party notices

The Active Light Optimizer code is MIT licensed (see LICENSE). This file credits the third-party work that the release builds on or refers to.

## 1. EnergyPlus

Some model parts are re-implemented in Python from the formulas documented for EnergyPlus. No EnergyPlus source code is included in this release.

- `o13.py`, `cw_model = "eplus_exact"`: the wind pressure coefficient Cw = 0.55 - 0.25 * angle / 45 for the open-area flow, re-implemented from the documented WindandStackOpenArea method (ZoneEquipmentManager).
- `thermal_hb2.py`: the SimpleGlazingSystem correlations (EnergyPlus Engineering Reference), re-implemented from the documented formulas.
- The bundled reference data `data/eplus_ref/A04_felix_open/ref.csv.gz` is output of EnergyPlus simulations of the author's own room (see section 2 for the weather input).

EnergyPlus licence text (reproduced as required):

---

EnergyPlus, Copyright (c) 1996-2024, The Board of Trustees of the University of Illinois, The Regents of the University of California, through Lawrence Berkeley National Laboratory (subject to receipt of any required approvals from the U.S. Dept. of Energy), Oak Ridge National Laboratory, managed by UT-Battelle, Alliance for Sustainable Energy, LLC, and other contributors. All rights reserved.

NOTICE: This Software was developed under funding from the U.S. Department of Energy and the U.S. Government consequently retains certain rights. As such, the U.S. Government has been granted for itself and others acting on its behalf a paid-up, nonexclusive, irrevocable, worldwide license in the Software to reproduce, distribute copies to the public, prepare derivative works, and perform publicly and display publicly, and to permit others to do so.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

(1) Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

(2) Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

(3) Neither the name of the University of California, Lawrence Berkeley National Laboratory, the University of Illinois, U.S. Dept. of Energy nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

(4) Use of EnergyPlus(TM) Name. If Licensee (i) distributes the software in stand-alone form without changes from the version obtained under this License, or (ii) Licensee makes a reference solely to the software portion of its product, Licensee must refer to the software as "EnergyPlus version X" software, where "X" is the version number Licensee obtained under this License and may not use a different name for the software. Except as specifically required in this Section (4), Licensee shall not use in a company name, a product name, in advertising, publicity, or other promotional activities any name, trade name, trademark, logo, or other designation of "EnergyPlus", "E+", "e+" or confusingly similar designation, without the U.S. Department of Energy's prior written consent.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

---

This project is not endorsed by or affiliated with the EnergyPlus copyright holders or the U.S. Department of Energy.

## 2. Weather data

The reference data `data/eplus_ref/A04_felix_open/ref.csv.gz` contains values derived from the Williamtown (Australia) TMYx EPW weather file, TMYx 2011-2025, station 947760.

- Source and credit: Climate.OneBuilding.Org, https://climate.onebuilding.org. Copyright (c) 2014-2026 Climate.OneBuilding. The site asks to cite: Lawrie, Linda K, Drury B Crawley. 2026. Development of Global Typical Meteorological Years (TMYx). The solar data in these files is based on ERA5 reanalysis (credited by the site to Oikolab).
- What the data provider states: a copyright notice (Copyright (c) 2014-2026 Climate.OneBuilding) and a request to cite the paper named above. No separate licence terms were found on the pages checked. If you reuse or redistribute this file, check the Climate.OneBuilding.Org site yourself. The Williamtown EPW file itself is included as an option in `optional/weather/Williamtown/` (README.txt there); it is used only if you point `epw` at it, and no weather file is set by default.

## 3. Methods and standards (cited, no text copied)

- Solar position: NOAA solar calculator equations, fractional-year form after Spencer (1971).
- Clear-sky irradiance: Meinel and Meinel clear-sky model.
- Door flow: Brown and Solvason doorway exchange, ASHRAE Handbook of Fundamentals (chapter on airflow around buildings).
- Elevated air speed: cooling offsets from ASHRAE Standard 55 (thermal environmental conditions for human occupancy).
- Adaptive comfort: EN 16798-1 category bands (running mean outdoor temperature).

## 4. Other tools

The Active Light Optimizer code itself is MIT licensed (see LICENSE). Ladybug Tools and Honeybee were used only as independent cross-checks during development; no code from them is included in this release.
