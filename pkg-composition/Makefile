.PHONY: clean
clean:
	rm -rf test-pkg

.PHONY: bootstrap
bootstrap:
	kpt pkg get git@github.com:michaelvl/kpt-test.git/pkgtst@idea/pkg-composition test-pkg
	cat test-pkg/Kptfile

.PHONY: render
render:
	kpt fn render . -o stdout
