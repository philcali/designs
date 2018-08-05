# Serverless Design Patterns

As the world moves incrementally towards serverless architecture, a
very common problem arises with batch processing. The problem:
Given a data source of unknown size, process the records at a given
cadence under the compute limitations (both size and time) of
serverless functions.

## Asynchronous Batch Processing of Indefinite Data Sizes

Strictly speaking in AWS, the solution is to break up the work in
sizable chunks and stream process the work as parallel as possible.

A very real world example is processing all of the stored RSS feeds
or crawling all known sources for new information. Here is what the
architecture would look like:

![Async Crawler Fanout][1]

1. CloudWatch is the timed incovation to initiate the workflow
1. A lambda performs a single page scan of the table in Dynamo
1. The lambda writes an SQS message with the next token (if available)
1. The same lambda iterates over the table until no more next tokens

Whatever work down in iteration, in this case is simply writing a
message to SQS, is fully parallelized, untilizing maximum concurrency
of the Lambda invocations for a given function. The work itself is
broken down quite substantially for the size constraints. For example:
a single function is responsible for reading the latest information
from a single RSS feed. The result of the work can be stored
permanently or strung together for more reactive workflows, like the
one below:

![Async Update Fanout][2]

In this case, new articles were added to a feed. This update stream
can kick off a similar workflow to iterate over all subscribers until
no more tokens exist. Once again, this work is fully parallelized,
and bypasses the need of SNS. Why is this important? SNS limits are
not indefinite. SNS is a great tool for sizable pub/sub workflows,
however passing the 12.5 million subscriber mark will force an
engineer to either:

- Creatively shard topics and subscribers
- Kindly ask AWS support for a limit increase

[1]: images/Async_Crawler_Fanout.png
[2]: images/Async_Update_Fanout.png
