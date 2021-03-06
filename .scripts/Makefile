# this Makefile is maintained inside the mara-app package: https://github.com/mara/mara-app/blob/master/.scripts/Makefile

SHELL=.scripts/makeshell $(or $@,-) # coloring + timing
.SILENT: # disable command echoing, will be done by .makeshell

SYSTEM_PYTHON36:=$(shell PATH=$(subst $(CURDIR)/.venv/bin:,,$(PATH)) which python3.6)

# update scripts from mara-app
update-scripts:=rsync -avq packages/mara-app/.scripts/Makefile ./Makefile; \
                rsync -avq packages/mara-app/.scripts/ensure-pushed.sh ./.scripts/ensure-pushed.sh; \
                rsync -avq packages/mara-app/.scripts/makeshell ./.scripts/makeshell;

# run auto-migrations
migrate:=FLASK_APP=app/app.py .venv/bin/flask mara_app.migrate

# install packages collected by pip freeze
install-packages: update-virtual-env check-for-uncommitted-changes
	.venv/bin/pip install --requirement=requirements.txt.freeze --src=./packages --upgrade
	$(call update-scripts)
	$(call migrate)
	echo -e "done. Get started with\n   $$ source .venv/bin/activate\n   $$ flask"

# create or update virtualenv
update-virtual-env: .venv/bin/python

.venv/bin/python: Makefile # rebuild virtuenv whenever Makefile changes
# if .venv is already a symlink, don't overwrite it
	mkdir -p .venv
# go into the new dir and build it there as venv doesn't work if the target is a symlink 
	cd .venv && $(SYSTEM_PYTHON36) -m venv --copies --prompt='[$(shell basename `pwd`)/.venv]' .
# set environment variables
	echo export FLASK_DEBUG=1 >> .venv/bin/activate
	echo export FLASK_APP=$(shell pwd)/app/app.py >> .venv/bin/activate
	echo export MAKEFLAGS=\"-j 4\" >> .venv/bin/activate
# add the project directory to path
	echo $(shell pwd) > `echo .venv/lib/*/site-packages`/mara-path.pth
	.venv/bin/pip install --upgrade pip setuptools pipdeptree wheel


update-packages: check-for-uncommitted-changes .venv/bin/python
	PYTHONWARNINGS="ignore" .venv/bin/pip install --requirement=requirements.txt --src=./packages --upgrade --process-dependency-links
	$(call update-scripts)
	.venv/bin/pipdeptree --warn=fail
	# pkg-ressources is automatically added on ubunut, but breaks the install.
	# https://stackoverflow.com/a/40167445/1380673
	.venv/bin/pip freeze | grep -v "pkg-resources" > requirements.txt.freeze
	$(call migrate)
	echo -e "\033[32msucceeded, please check output above for warnings\033[0m"

update-scripts:
	$(call update-scripts)

migrate:
	$(call migrate)


pipdeptree:
	.venv/bin/pipdeptree --warn=fail

clean:
	rm -rf .venv/*
	rm -rf packages/*



check-for-uncommitted-changes: $(addprefix .ensure-pushed-,$(subst ./,,$(shell mkdir -p packages; cd packages; find . -maxdepth 1 -mindepth 1 -type d)))

.ensure-pushed-%:
	.scripts/ensure-pushed.sh packages/$*


