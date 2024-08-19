---
layout: single
title:  "Lumenalta Technical Assessment"
date:   2024-08-18 10:24:38 -0600
categories: interview lumenalta
---

In March 2024, I got an email from someone at [Lumenalta](https://lumenalta.com/) with the link to the technical assessment. It stated that I would have 2 hours to complete the assessment which was real-world coding (**so excited**).

There are two parts to the assessment. The first part is related to the front-end application. It required adding new features, modifying the existing ones, and even updating the unit test. The second part is related to back-end development. In this article, I would like to share what I have done so far on this assessment even though I couldn't get it done in 2 hours (:sob::sob::sob:). Btw, it was good to have a chance to do the assessment :smile:.

### Let's jump right into it.
The title of the assessment was called "Javascript: Parallel Asynchronous Programming". It means that you have to create something that is gonna run **asynchronous** and **parallelly**. 

### The objective
Your goal is to write an asynchronous function `pooledDownload`. This function needs to be able to parallelly download and save files from a given array of URLs, using a connection pool of a given size.


### The interesting part was the acceptance criteria
* All files need to be downloaded and saved.
* File contents should be saved as soon as they are downloaded.
* Downloads must be distributed over the connections as evenly as possible.
* One connection can only be downloading one file at a time. You'll need to create more connections to download multiple files at the same time.
* Any opened connections must always be closed.
* If no connections could be opened to the server, your function needs to reject with a new `Error` containing the message `"connection failed"`.
* If any error occurred during the transfer, your function needs to reject with this same error.
* BONUS: Sometimes, the server may not have enough slots to accept the requested amount of concurrent connections. In this case, you are to stop opening new connections once the server reaches its capacity and proceed with as many connections as the server allows.
* You can choose whether to use `Promise`s, `async`/`await` or a mixture of them

At first, I tried to solve it by using `Promise` but I couldn't get it. Then I switched to `async/await` and passed all the unit tests.

The code below reflects what I was able to accomplish after the time expired. I wish I had more time or another opportunity to revisit it, as I found the task truly engaging.

```javascript
/**
 * downloads were distributed evenly among the successful connections
 * @param {*} numberOfRecipients
 * @param {*} totalAmount
 * @returns Array
 *
 * Example: if number of recipients is 4, and totalAmount is 10
 *  -> Output: [3, 3, 2, 2]
 */
const distributeEvenly = (numberOfRecipients, totalAmount) => {
  const min = parseInt(totalAmount / numberOfRecipients);
  const remainder = totalAmount % numberOfRecipients;
  return Array(numberOfRecipients)
    .fill(0)
    .map((_, idx) => min + (idx < remainder ? 1 : 0));
};

const pooledDownload = async (connect, save, downloadList, maxConcurrency) => {
  const connections = [];

  try {
    for (let i = 0; i < maxConcurrency; i++) {
      connections.push(await connect());
    }
  } catch (_) {
    /** Ignore error & stop calling connect to server */
  }

  if (connections.length === 0) {
    throw new Error("connection failed");
  }

  const fileDistributed = distributeEvenly(
    connections.length,
    downloadList.length
  );

  return Promise.all(
    connections.map((connection, idx) => {
      const { download } = connection;
      const count = fileDistributed[idx];
      const files = downloadList.splice(0, count);
      return Promise.all(
        files.map((file) => {
          return download(file).then((result) => save(result));
        })
      );
    })
  ).finally(() => connections.forEach((conn) => conn.close()));
};

module.exports = pooledDownload;
```

Hope you all enjoy it and good luck if you found one with your search in 2024. In the next post, I will share my journey with another technical assessment which almost took me for three days.