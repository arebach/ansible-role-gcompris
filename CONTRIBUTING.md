## Overview

Thank you for your interest in contributing to the GCompris RPi Kiosk Ansible role! This document covers everything you need to know to contribute effectively.

## Prerequisites

- Ansible 2.15+
- `ansible-lint` installed (`pip install ansible-lint`)
- Docker (for Molecule tests)
- A Raspberry Pi for real-world testing (optional but recommended)

## Development Setup

```bash
git clone https://github.com/arebach/ansible-role-gcompris.git
cd ansible-role-gcompris
```

## Project Structure

```
ansible-role-gcompris/
├── galaxy.yml                    # Galaxy metadata
├── CHANGELOG.md                 # Version history
├── playbooks/
│   ├── provision.yml             # Deployment playbook
│   └── diagnostics.yml           # Boot chain health check
├── roles/arebach/gcompris_rpi_kiosk/
│   ├── defaults/main.yml         # User-overridable variables
│   ├── vars/main.yml             # Internal variables (package lists)
│   ├── tasks/main.yml            # Main provisioning tasks
│   ├── tasks/network-*.yml       # Network mode task files
│   ├── handlers/main.yml         # Service restart handlers
│   ├── templates/                # Jinja2 templates
│   ├── meta/                     # Galaxy metadata, requirements, runtime
│   └── molecule/default/         # Molecule test suite
└── .github/                     # CI/CD, issue/PR templates
```

## Coding Standards

### ansible-lint

All code must pass `ansible-lint` with the **production** profile:

```bash
ansible-lint --profile production
```

This enforces:
- FQCN for all modules (`ansible.builtin.apt`, not `apt`)
- No `command` where a module exists
- Proper `changed_when`/`failed_when` on shell tasks
- `pipefail` on piped shell commands
- Variable naming (role-prefixed)

### Task Naming

- Use descriptive task names with section comments
- Prefix variables with `gcompris_rpi_kiosk_`
- Use `true`/`false` (not `yes`/`no`) for booleans

### Templates

- All config files that use variables must be Jinja2 templates (`.j2`)
- Static config files go in `files/` (if any)
- Include deployment path in template header comments

### Defaults vs Vars

- `defaults/main.yml` — user-overridable variables with documentation comments
- `vars/main.yml` — internal variables (package lists, service names) that users shouldn't typically change

## Testing

### Lint

```bash
ansible-lint --profile production
```

Must pass with 0 failures and 0 warnings.

### Molecule (CI)

```bash
cd roles/arebach/gcompris_rpi_kiosk
molecule test
```

Molecule tests verify static configuration file generation in a Docker container. They do NOT test runtime behavior (e.g., whether GCompris actually launches).

### Real Pi Testing

For changes that affect runtime behavior:

1. Point your inventory at a real Raspberry Pi
2. Run the full playbook: `ansible-playbook -i inventory.local.yml playbooks/provision.yml`
3. Verify GCompris launches after reboot
4. Run diagnostics: `ansible-playbook -i inventory.local.yml playbooks/diagnostics.yml`

### Diagnostics

The diagnostics playbook checks all 12 links in the boot-to-launch chain:

```bash
ansible-playbook -i inventory.local.yml playbooks/diagnostics.yml
```

All checks should show ✅ PASS.

## Pull Request Process

1. **Fork** the repository and create your branch from `main`
2. **Branch naming:** `feat/`, `fix/`, `docs/`, `ci/`, `refactor/`
3. **Commit style:** Use [conventional commits](https://www.conventionalcommits.org/):
   - `feat: add support for custom GCompris config file`
   - `fix: resolve LightDM autologin race condition`
   - `docs: update README with new variable`
4. **Test:** Ensure `ansible-lint --profile production` passes
5. **Document:** Update README.md and CHANGELOG.md if adding/changing variables or behavior
6. **PR template:** Fill out the pull request template completely
7. **Review:** Address any feedback from code review

### Variable Changes

If you add or change a variable:

1. Add to `defaults/main.yml` with documentation comment
2. Update README.md Variables Reference section
3. Update CHANGELOG.md
4. Add to molecule `converge.yml` if it needs testing

### Template Changes

If you add or change a template:

1. Ensure it's a `.j2` file in `templates/`
2. Update molecule `verify.yml` to check the generated content
3. Update README.md Project Structure if adding a new file

## Release Process

Releases are managed by the maintainer (arebach):

1. Version bumped in `galaxy.yml`
2. CHANGELOG.md updated with release notes
3. Tag created: `git tag v1.x.0`
4. GitHub release created
5. Ansible Galaxy auto-publishes via CI/CD

## Code of Conduct

Be respectful, constructive, and welcoming. We're building educational tools for children — let's model that spirit in our collaboration.

## Questions?

Open a [support question issue](https://github.com/arebach/ansible-role-gcompris/issues/new?template=support_question.yml) or start a [discussion](https://github.com/arebach/ansible-role-gcompris/discussions).
