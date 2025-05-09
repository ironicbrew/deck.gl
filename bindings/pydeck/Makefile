BIN=env/bin/

help:
	@echo "\nPlease use \`make target\` where target is one of\n"
	@echo "init - build deck.gl and install all requirements for development"
	@echo "test - run tests"
	@echo "prepare-jupyter - initiate pydeck-related extensions in JupyterLab and Jupyter Notebook"
	@echo "bump-version - bump the version files for the pydeck frontend and backend"
	@echo "publish-pypi - prepare pydeck for publication to PyPI"
	@echo "publish-test-pypi - prepare pydeck for publication to the test.pypi.org"
	@echo "release - package and upload a release"
	@echo "fast-build - rebuild @deck.gl/jupyter-widget"
	@echo "setup-env - detect basic requirements for building the library and set up a virtualenv"
	@echo "pre-commit - run pre-commit hook for pydeck"

init:
	. $(BIN)/activate; \
	yarn bootstrap; \
	yarn build; \
	$(BIN)pip install -r requirements/requirements.txt; \
	$(BIN)pip install -r requirements/requirements-dev.txt; \
	python setup.py develop

fast-build:
	cd ../../modules/jupyter-widget; webpack; cd -; \
	python setup.py develop

pre-commit:
	python -m black . --exclude=env --target-version py34 && $(MAKE) test

setup-env:
	test `command -v python3` || echo Please install python3
	test `command -v yarn` || echo Please install yarn
	test `command -v python3` && test `command -v yarn`
	python3 -m venv env
# 	avoid conflict between any js code in the env that in the root
	echo '{"type": "commonjs"}' > env/package.json

prepare-jupyter:
	# Enable the notebook extension
	python -m jupyter nbextension install --sys-prefix --symlink --overwrite --py pydeck; \
	python -m jupyter nbextension enable --sys-prefix --py pydeck; \
	# Build and install the JS module
	cd ../../modules/jupyter-widget/; \
	python -m jupyter labextension install @jupyter-widgets/jupyterlab-manager --no-build; \
	python -m jupyter labextension install . --no-build; \
	python -m jupyter labextension link .; \
	python -m jupyter lab build; \
	cd ../../bindings/pydeck

test:
	$(BIN)python -m flake8 . && \
	$(BIN)python -m pytest --cov=pydeck

bump-version:
	@read -p "Choose a release type {MAJOR, MINOR, PATCH, BETA, ALPHA, RC}:" choice; \
	python bump_version.py "$$choice";

publish-pypi:
	rm -rf ./dist/*
	python setup.py sdist bdist_wheel
	pip install twine
	python -m twine upload dist/*

publish-test-pypi:
	rm -rf ./dist/*
	python setup.py sdist bdist_wheel
	pip install twine
	python -m twine upload --repository-url https://test.pypi.org/legacy/ dist/*

release:
	$(MAKE) bump-version;
	$(MAKE) test;
	VERSION=$$(python -c "import pydeck; print(pydeck.__version__)");
	git commit -am "pydeck $$VERSION";
	$(MAKE) publish-pypi;

ci:
	$(MAKE) test
