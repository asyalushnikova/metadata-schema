## Schema test files

To add a JSON file to automatic Travis CI testing, complete the following:

1. Branch off of `develop`.
1. Under the `schema_test_files` folder, find the appropriate entity type for the JSON file example you want to add and navigate there.
1. Add the test JSON file to this directory
1. Update `src/json_examples_validate_against_schema.py`
   1. Add a block that tests the new JSON file
   1. Add the appropriate schema against which validation should occur
   1. If the new JSON file is meant to fail, indicate what the failing error message should say
1. Test `json_examples_validate_against_schema.py` locally. 
    1. If it passes, create a PR with the changes
    1. If it doesn't pass, debug.
