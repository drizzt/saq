# Jobs

Jobs can be scheduled to run as:
* Fire-and-forget tasks: {py:class}`saq.queue.Queue.enqueue`
* Wait-for-result tasks: {py:class}`saq.queue.Queue.apply`

Sample code:
```python
# schedule a job normally
job = await queue.enqueue("test", a=1)

# wait 1 second for the job to complete
await job.refresh(1)
print(job.results)

# run a job and return the result
print(await queue.apply("test", a=2))

# schedule a job in 10 seconds
await queue.enqueue("test", a=1, scheduled=time.time() + 10)
```

## Common

### Explicit vs Implicit job calling
TODO: queue.enqueue(Job(...)) vs known-parameters

### Job defaults
TODO: Discuss example to subclass Queue to configure defaults for enqueue (e.g. retries)

### Retries
TODOL Discuss that retries ALWAYS jitter

### Continuation — draining large jobs in chunks

A handler that has made partial progress on a bounded chunk of work can
request that the job be re-queued under the *same key* by calling
{py:meth}`saq.job.Job.requeue`. The worker commits the re-queue after the
handler returns cleanly, resetting `attempts` so the continuation counts as
a fresh run.

Unlike {py:meth}`saq.job.Job.retry`, `requeue` is not triggered by a failure
and does not replace existing retry semantics. Use it when the job is
making progress but needs another iteration — for example a rate-limited
API whose per-call quota forces the handler to stop short of completion.

```python
async def drain(ctx, *, chat_id: int, cursor: int = 0):
    processed, next_cursor = await drain_chunk(chat_id, cursor)
    if next_cursor is not None:
        await ctx["job"].requeue(cursor=next_cursor)
    # else: handler returns, job finishes normally.
```

If the handler raises, is aborted, or exceeds its timeout, the requeue
request is discarded and SAQ's standard retry / fail / abort paths apply.

## Enqueue
```{eval-rst}
.. autoapifunction:: saq.queue.Queue.enqueue
    :noindex:
```

## Apply
```{eval-rst}
.. autoapifunction:: saq.queue.Queue.apply
    :noindex:
```

## Map
```{eval-rst}
.. autoapifunction:: saq.queue.Queue.map
    :noindex:
```

## Batch
```{eval-rst}
.. autoapifunction:: saq.queue.Queue.batch
    :noindex:
```
