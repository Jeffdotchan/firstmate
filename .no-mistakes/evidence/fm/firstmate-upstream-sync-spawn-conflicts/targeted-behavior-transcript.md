# Firstmate upstream-sync targeted behavior transcript

Target tested: `b0b8288ec1662dae64ad248c791d2b74f282313a`, plus the test-phase Bash 3.2 portability fix recorded in the worktree.

## Merged spawn conflict behavior

Command:

```console
$ FM_TEST_EVIDENCE=1 bash tests/fm-spawn-pool-base-freshen.test.sh
# observed local-only no-origin spawn: spawned pool-local-only-no-origin-r6 harness=codex kind=ship mode=local-only yolo=off window=firstmate:fm-pool-local-only-no-origin-r6 worktree=<fixture>/local-only-no-origin/pool
# observed base: HEAD=d608cad914962a20bb09bc24e8f5b86818033f9a primary=d608cad914962a20bb09bc24e8f5b86818033f9a advanced=d608cad914962a20bb09bc24e8f5b86818033f9a
ok - a local-only no-origin pooled worktree refreshes from the primary local checkout
ok - a local-only no-origin scout refreshes from the primary local checkout
# observed stale-pin refusal: error: submodule 'ui' is checked out at 6be864f302f058ee9b2bce352476a9e7f8553da3, but this base records 5147aacb1a50d2955128e26d06f639595f20b086
ok - two consecutive spawns across a moved submodule pin end in a refusal naming both pins and no remedy
# all fm-spawn-pool-base-freshen tests passed
```

The executable suite ran all 15 merged cases. The transcript shows both conflict sides surviving: local-only/no-origin spawn uses the verified primary tip, while a stale submodule checkout is refused with both pins named.

## Bounded teardown failure mode

The focused `test_hung_lsof_scan_is_bounded_and_refuses` case invoked the real `bin/fm-teardown.sh` with an `lsof` stand-in that sleeps for 600 seconds:

```console
ok - a hung system-wide lsof scan is bounded by wall clock and refuses instead of hanging
# observed teardown exit: bounded hung-lsof path completed in 5s
# observed operator-facing stderr:
REFUSED: cannot determine leaked processes under <fixture>/lsof-hang-bounded/wt for task-x1 (lsof failed); preserving the worktree/tasktmp for manual inspection or retry.
# observed preserved state: worktree=present metadata=present returned=no
```

## Remote retirement cleanup

The first remote lifecycle run exposed the product's Bash 3.2 `set -u` failure for project-less remote seeds:

```console
bin/fm-remote-home-seed.sh: line 190: PROJECT_NAMES[@]: unbound variable
```

After changing the empty-array loop to the repository's portable expansion, the full remote lifecycle e2e passed under the host's supported `C` locale:

```console
$ LC_ALL=C LANG=C LC_CTYPE=C FM_TEST_EVIDENCE=1 bash tests/fm-remote-secondmate-lifecycle-e2e.test.sh
# observed remote retirement: home=absent metadata=absent reconcile-cooldown=absent sibling-workspace=present shared-session=running
ok - remote retirement refuses child work, then removes only its own endpoint while a shared-session sibling survives
ALL TESTS PASSED
```
