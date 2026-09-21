---
title: "Django With Multiprocessing"
summary: "Django With Multiprocessing"
tags: ["django"]
date: 2026-09-19
draft: true
showToc: true
TocOpen: true
lightCode: true
comments: true
---

If you use Django with multiprocessing, you should be careful that data loss is possible if this is not done properly.
The problem is that the connection Django makes to the database, is shared if you start multiprocessing naively.
Sharing this connection between all processes can cause the data loss.
That is why if you use multiprocessing, close the connection.
Whenever Django then needs a connection it will make a new one in that process which is then not shared.

```py
def process(chunk):
    try:
        db.connections.close_all()

        with db.transaction.atomic():
            process_patient_folder(chunk)
    except Exception as err:
        with ERROR_FILE.open("a") as f:
            print("ERROR", err, chunk, file=f)
            traceback.print_exception(err, file=f)
        raise


def main(chunks):
    db.connections.close_all()
    with mp.Pool(nproc) as pool:
        pool.map(process, chunks)
```

This is also how Celery does its multiprocessing.
