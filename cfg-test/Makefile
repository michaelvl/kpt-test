# Git commits
INITIAL_NO_FUNCTION_NAMES = 4f5f9d2
ADDED_FUNCTION_AT_END_NO_FUNCTION_NAMES = 31675d8
ADDED_FUNCTION_AT_FRONT_NO_FUNCTION_NAMES = a452d7b
ADDED_FUNCTION_NAMES = b07ab69
ADDED_ANOTHER_NAMED_FUNCTION = d60dafa

.PHONY: clean
clean:
	rm -rf test-pkg

.PHONY: bootstrap
bootstrap:
	kpt pkg get git@github.com:michaelvl/kpt-test.git/pkgtst@$(INITIAL_NO_FUNCTION_NAMES) test-pkg
	cat test-pkg/Kptfile

.PHONY: update
update:
	kpt pkg update test-pkg@$(ADDED_FUNCTION_AT_FRONT_NO_FUNCTION_NAMES)
	cat test-pkg/Kptfile

.PHONY: reset-back
reset-back:
	kpt pkg update test-pkg@$(INITIAL_NO_FUNCTION_NAMES)
	cat test-pkg/Kptfile

.PHONY: reset-back-force-delete-replace
reset-back-force-delete-replace:
	kpt pkg update test-pkg@$(INITIAL_NO_FUNCTION_NAMES) --strategy force-delete-replace
	cat test-pkg/Kptfile


.PHONY: bootstrap-named
bootstrap-named:
	kpt pkg get git@github.com:michaelvl/kpt-test.git/pkgtst@$(ADDED_FUNCTION_NAMES) test-pkg
	cat test-pkg/Kptfile

.PHONY: update-named
update-named:
	kpt pkg update test-pkg@$(ADDED_ANOTHER_NAMED_FUNCTION)
	cat test-pkg/Kptfile

.PHONY: reset-back-named
reset-back-named:
	kpt pkg update test-pkg@$(ADDED_FUNCTION_NAMES)
	cat test-pkg/Kptfile

.PHONY: reset-back-force-delete-replace-named
reset-back-force-delete-replace-named:
	kpt pkg update test-pkg@$(ADDED_FUNCTION_NAMES) --strategy force-delete-replace
	cat test-pkg/Kptfile
