# Getting started

This walkthrough presents what you can do with OpenFisca without installing it.

See the [getting started Jupyter Notebook](https://github.com/openfisca/openfisca-france/blob/master/notebooks/getting-started.ipynb) which illustrates this section.

## Explore the legislation

The [legislation explorer](http://legislation.openfisca.fr/) allows you to browse the variables and the parameters of the tax and benefit legislation.

Each variable has its own URL and page, and the same for the parameters.

For example, the "allocations familiales": [af](http://legislation.openfisca.fr/variables/af)

## Calculate a variable

### Using Python

Say we want to calculate the [af](http://legislation.openfisca.fr/variables/af) of a family with 1 parent (they divorced) and 3 children.

Here would be the script:

```python
import openfisca_france

tax_benefit_system = openfisca_france.FranceTaxBenefitSystem()

scenario = tax_benefit_system.new_scenario()
scenario.init_single_entity(
    period = 2015,
    parent1 = dict(
        age = 30,
        salaire_de_base = 15000,
        ),
    enfants = [
        dict(age = 10),
        dict(age = 12),
        dict(age = 18),
        ],
    )

simulation = scenario.new_simulation()
af = simulation.calculate('af', '2015-01')
print af
```

Result:

```
[ 361.52359009]
```

The result is a vector of size 1, the number of `families` in our test case.

On the line `af = simulation.calculate('af', '2015-01')`, `'2015-01'` corresponds to a period (january 2015).

> OpenFisca uses periods intensively so it defines string shortcuts to express them.

### Using the web API

Let's do the same calculation using the web API hosted by the OpenFisca project.

The web API is callable from any language which supports HTTP requests and JSON.
We could write our example in Python or JavaScript, but let's use [curl](http://curl.haxx.se/)
and [jq](https://stedolan.github.io/jq/).

```bash
read -d '' json << EOF
{
  "scenarios": [
    {
      "test_case": {
        "familles": [
          {
            "parents": ["parent1"],
            "enfants": ["enfant1", "enfant2", "enfant3"]
          }
        ],
        "foyers_fiscaux": [
          {
            "declarants": ["parent1"],
            "personnes_a_charge": ["enfant1", "enfant2", "enfant3"]
          }
        ],
        "individus": [
          {
            "id": "parent1",
            "age": 30,
            "salaire_de_base": {"2015": 15000}
          },
          {
            "id": "enfant1",
            "age": 10
          },
          {
            "id": "enfant2",
            "age": 12
          },
          {
            "id": "enfant3",
            "age": 18
          }
        ],
        "menages": [
          {
            "personne_de_reference": "parent1",
            "enfants": ["enfant1", "enfant2", "enfant3"]
          }
        ]
      },
      "period": "2015-01"
    }
  ],
  "output_format": "variables",
  "variables": ["af"]
}
EOF
curl -H "Content-Type: application/json" -X POST -d "$json" http://api.openfisca.fr/api/1/calculate | jq .value[0]
```

Result:

```json
{
  "af": {
    "2015-01": [
      361.5235900878906
    ]
  }
}
```

This is sightly more verbose than the previous example because we express all the entities explicitly,
whereas in Python we had the helper method `init_single_entity`.
That's because most of the time the JSON payload of an HTTP request is generated programatically.

## Test the impact of a reform

OpenFisca can be used to test the impact of a reform. For the purpose of this tutorial we will work on a test case,
but it's possible to work on population data like surveys.

Let's use a reform which has already been coded: the .
A dedicated section of the documentation explains how to code a reform.

Here is the script `test2.py` available
[here](https://github.com/openfisca/openfisca-france/tree/master/openfisca_france/scripts/getting_started/test2.py)
or in the [getting started Jupyter Notebook]:

```python
from openfisca_france.reforms import landais_piketty_saez

reform = landais_piketty_saez.landais_piketty_saez(tax_benefit_system)

def init_profile(scenario):
    scenario.init_single_entity(
        period = '2013',
        parent1 = dict(
            age = 40,
            salaire_de_base = 50000,
            ),
        )
    return scenario

reform_scenario = init_profile(reform.new_scenario())
reform_simulation = reform_scenario.new_simulation() 

print (reform_simulation.calculate('revdisp', '2013'))
```

Result:

```
[ 15614.34863281]
```

The result is a vector of size 1, the number of `foyers_fiscaux` in our test case.