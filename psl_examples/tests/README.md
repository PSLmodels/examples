# How to update the files for tests of new `taxcalc` reform files

When adding a Tax-Calculator reform file, there are a few steps you'll need to take to update the repository for the testing suite to continue to pass.  The `test_taxcalc_reforms.py` module reads all of the reform JSON files in `Examples/psl_examples/taxcalc` (even your new one) and runs Tax-Calculator using the reform file and the ``Examples/psl_examples/taxcalc/cases.csv` file.  It then looks for a file in the `Examples/psl_examples/taxcalc` directory with the name `YOUR_REFORM.out.csv`.  For the tests to pass, this file will need to be present and the results in that file will need to match what one gets when running your reform through Tax-Calculator using the `cases.csv` as the data file.

To create this file, build and enter the `psl-examples-env` Conda environment using the `environment.yml` file in this repository.  Be careful to make sure you are running the lastest version of Tax-Calculator, installed from the master branch in the [Tax-Calculator repository](https://github.com/PSLmodels/Tax-Calculator/) (the environment file will handle this for you, but once created, you'll want to keep your environment up-to-date).  Next, you can do the following in the Python script or interactive Python session (this code snippet assumes you are in the ``Examples/psl_examples/tests` directory, update paths accordingly if you are not):

```
import pandas as pd
import taxcalc
pol = taxcalc.policy.Policy()
rec = taxcalc.records.Records(
    data='../taxcalc/cases.csv', start_year=2020, gfactors=None,
    weights=None, adjust_ratios=None)
base_calc = taxcalc.calculator.Calculator(policy=pol, records=rec)
ref = pol.read_json_reform('../taxcalc/YOUR_REFORM.json')
pol.implement_reform(ref)
reform_calc = taxcalc.calculator.Calculator(policy=pol, records=rec)
reform_calc.advance_to_year(2020)
reform_calc.calc_all()
df = reform_calc.dataframe(['RECID', 'c00100', 'standard', 'c04800', 'iitax', 'payrolltax'])
df.to_csv('../taxcalc/YOUR_REFORM.out.csv', index=False, float_format='%.2f')
```

Note that the start year is assumed to be 2020.  This is hardcoded into `test_taxcalc_reforms.py`.  But if your reform has not changes to tax law in calendar year 2020, then you will also need to modify the `test_taxcalc_reforms.res_and_out_are_same()` function to not use a hardcoded year, but rather use the first year your reform file specifies changes from current law.