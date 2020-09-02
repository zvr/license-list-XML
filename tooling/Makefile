# SPDX-License-Identifier: CC0-1.0

need := 3.81
ok := $(filter $(need),$(firstword $(sort $(MAKE_VERSION) $(need))))
ifndef ok
	$(error Need GNU make at least v3.81)
endif

assert-command-present = $(if $(shell which $1),,$(error '$1' missing and needed for this build))
$(call assert-command-present,curl)
$(call assert-command-present,gpg)
$(call assert-command-present,git)
$(call assert-command-present,java)
$(call assert-command-present,md5sum)


add_checksum_files = $1 $(addsuffix .md5,$1)


INPUT_DIR = ../src
ALL_LICENSES = $(wildcard $(INPUT_DIR)/*.txt)
ALL_EXCEPTIONS = $(wildcard $(INPUT_DIR)/exceptions/*.txt)
ALL_INPUTS = $(ALL_LICENSES) $(ALL_EXCEPTIONS)

TOOL_VERSION = 2.1.22

TEST_DATA = test/simpleTestForGenerator
GIT_AUTHOR = License Publisher (maintained by Gary O'Neall) <gary@sourceauditor.com>
LICENSE_DATA_REPO_NO_SCHEME = github.com/spdx/license-list-data.git
LICENSE_DATA_URL = https://$(GITHUB_TOKEN)@$(LICENSE_DATA_REPO_NO_SCHEME)

LICENSE_OUTPUT_DIR = .tmp
GITVERSION = $(shell git describe --always || echo 'UNKNOWN')
# Remove leading 'v' or 'V'
VERSION = $(subst V,,$(subst v,,$(GITVERSION)))
RELEASE_DATE = $(shell date '+%Y-%m-%d')
COMMIT_MSG = License list build $(VERSION) using license list publisher $(TOOL_VERSION)
RELEASE_MSG = Adding release matching the license list XML tag $(VERSION)

.PHONY: validate-canonical-match
validate-canonical-match: licenseListPublisher-$(TOOL_VERSION).jar-valid $(TEST_DATA) $(LICENSE_OUTPUT_DIR)
	java -jar -DLocalFsfFreeJson=false -DlistedLicenseSchema="schema/ListedLicense.xsd" licenseListPublisher-$(TOOL_VERSION).jar LicenseRDFAGenerator src '$(LICENSE_OUTPUT_DIR)' 1.0 2000-01-01 $(TEST_DATA) expected-warnings

.PHONY: deploy-license-data
deploy-license-data: licenseListPublisher-$(TOOL_VERSION).jar-valid $(TEST_DATA)
	rm -rf '$(LICENSE_OUTPUT_DIR)'
	git clone --quiet --depth 1 $(LICENSE_DATA_URL) '$(LICENSE_OUTPUT_DIR)'
	# Clean out the old data directories
	find '$(LICENSE_OUTPUT_DIR)' -mindepth 1 -maxdepth 1 -name .git -prune -o -type d -exec rm -rf {} \+
	rm -f $(LICENSE_OUTPUT_DIR)/licenses.md
	java -jar -DLocalFsfFreeJson=false -DlistedLicenseSchema="schema/ListedLicense.xsd" licenseListPublisher-$(TOOL_VERSION).jar LicenseRDFAGenerator src '$(LICENSE_OUTPUT_DIR)' $(VERSION) $(RELEASE_DATE) $(TEST_DATA) expected-warnings
	git -C '$(LICENSE_OUTPUT_DIR)' add -A .
	git -C '$(LICENSE_OUTPUT_DIR)' commit --author "$(GIT_AUTHOR)" -m "$(COMMIT_MSG)"
	git -C '$(LICENSE_OUTPUT_DIR)' push origin

.PHONY: release-license-data
release-license-data: deploy-license-data
	if [[ $(VERSION) =~ .+-g[a-f0-9]{7} ]] ; then echo Can not release license data - license list version $(VERSION) does not match a release pattern ;  	exit 1 ; else ; git -C '$(LICENSE_OUTPUT_DIR)' tag -a $(VERSION) -m "$(RELEASE_MESSAGE)" ; git -C '$(LICENSE_OUTPUT_DIR)' push --tags --quiet origin ; fi

.PRECIOUS: licenseListPublisher-%.jar
licenseListPublisher-%.jar:
	curl -L https://dl.bintray.com/spdx/spdx-tools/org/spdx/licenseListPublisher/$*/licenseListPublisher-$*-jar-with-dependencies.jar >$@

.PRECIOUS: licenseListPublisher-%.jar.asc
licenseListPublisher-%.jar.asc:
	curl -L https://dl.bintray.com/spdx/spdx-tools/org/spdx/licenseListPublisher/$*/licenseListPublisher-$*-jar-with-dependencies.jar.asc >$@

.PHONY: licenseListPublisher-%.jar-valid
licenseListPublisher-%.jar-valid: licenseListPublisher-%.jar.asc licenseListPublisher-%.jar goneall.gpg
	gpg --verify --no-default-keyring --keyring ./goneall.gpg $<

$(LICENSE_OUTPUT_DIR):
	mkdir -p $@

resources:
	mkdir -p $@

.PHONY: clean
clean:
	$(RM) -r $(LICENSE_OUTPUT_DIR)

.PHONY: clobber full-clean
clobber full-clean: clean
	$(RM) -r resources licenseListPublisher-*.jar*

%.md5: FORCE
	 @$(eval CHECKSUM := $(shell md5sum $*))$(if $(filter-out $(shell cat $@ 2>/dev/null),$(CHECKSUM)),echo $(CHECKSUM) > $@)

