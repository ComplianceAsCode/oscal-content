# Test to verify the quality of OSCAL content

This test ensures that updates to OSCAL content maintain its quality.

Given our new experience with this test, numerous discussions arose during its development. Discussions primarily focus on the scope and methodology of testing. All related discussions can be found in the comment section of [CPLYTM-254](https://issues.redhat.com/browse/CPLYTM-254). 

## Test workflow 

This test is triggered by a `pull request` event in [github workflow](../.github/workflows/quality-test.yml)

What this workflow do:

- Setup [complyctl](https://github.com/complytime/complyctl) test environment.
- Get a list of files changed by pull request, filtered according to what need to tested.
- Group changed files into `catalogs`, `profiles`, `component-definitions`.
- Call [verify-OSCAL-contents.sh](../scripts/verify-OSCAL-contents.sh) with correct parameters to verify changed OSCAL files.

Currently, there is only one job for testing `cusp_fedora` related OSCAL content. Other operating system's OSCAL content and tests can be easily included on-demand.

## verify-OSCAL-contents.sh script

This script validates OSCAL content (Catalog, Profile, and Component Definition) by using complyctl and OpenSCAP to ensure alignment between OSCAL rules and OpenSCAP rules.

usage: 
```shell
./scripts/verify-OSCAL-contents.sh "$catalogs" "$profiles" "$component_definitions" "$product_name"

# OSCAL Catalogs relative path string, multiple elements separate by space
catalogs=$1
# OSCAL Profiles relative path string, multiple elements separate by space
profiles=$2
# OSCAL Component Definitions relative path string, multiple elements separate by space
component_definitions=$3
# test product name 
product=$4  
```

### What this script do:

- Running complyctl `list`, `plan`, and `generate` commands using changed OSCAL contents.
- Calculate rule alignment between OSCAL and OpenSCAP, if alignment percentage lower than 60%, script will fail.

### How to calculate the rule alignment:

The script gets the number of rules from `assessment-plan.json` and the number of selected and unselected entries from `tailoring_policy.xml`, then calculate the % of rule alignment`((rule_count_oscal - unselected_rule) / rule_count_oscal) * 100`.

## How to add test for other product

- Add a new job in github workflow by copying the job `quality-test-fedora`
- Modify `container` field for target OS
- Modify step `Get all changed files`, change `files` parameter to filter what OSCAL files need to test.
For catalogs, you can just set `catalogs/**` because script will search `catalog` related files with `product` parameter.
- Modify last step `Test changed files`, change the last `product` parameter to what you test, for example [fedora](https://github.com/ComplianceAsCode/oscal-content/blob/main/component-definitions/fedora/fedora-cusp_fedora-default/component-definition.json#L14) 

## Workflow run example 

https://github.com/ComplianceAsCode/oscal-content/pull/92

