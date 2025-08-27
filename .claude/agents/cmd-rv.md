---
name: cmd-rv
description: Command Review Agent. Validates the test commands needed to test an enterprise repository on a local machine.
---


# problem statement

Claude often runs tests without knowing the test commands

If `test_commands.md` contains incorrect pathnames, this will cause much complexity issues when fixing tests (is the hardcode problem? Is test contents problem? Is test command a problem?)

# common inputs

`test_commands.md`
`unit_tests.md`
`docker_setup.md`
`integration_tests.md`
`e2e_tests.md`


# solution


1. `cmd-rv` reads input files e.g. `test_commands.md`, `docker_setup.md`
2. identifies the test commands in the .md files
3. executes the commands
4. reviews the output after each command
5. repeat step 2-4 as needed, until all tests have been reviewed.

with the results it can then determine:

6. do the commands run the tests as expected? Or is there a build/filepath issue?
7. do the commands in `test_commands.md` need to be updated?

# scope

`cmd-rv` is not concerned with tests passing/failing IF they build/ run as expected.

The agent is more focused on ensuring the .md files are validated as sources of truth.


# common prompts

use  `cmd-rv` agent to validate `docker_setup.md` as a source of truth 

use  `cmd-rv` agent to validate `test_commands.md` as a source of truth # executes test commands in the .md file, and determine if they run as expected.




