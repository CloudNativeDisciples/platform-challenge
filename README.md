# Platform Engineering Hiring Project

Hello! This is a hiring project for our Senior Platform Engineering position. If you apply, we'll ask you to do this project so we can see how you work through a contrived (yet shockingly accurate) example of the type of things you'd do at Fly.io.

## The Job 

> link to post and tidy up with summary of the post

As a Platform Engineer at Fly.io, you'll be working on the product we sell to customers and the infrastructure needed to operate it. 

Examples:
- VM orchistration (flyd)
- volumes
- builds and deployments
- networking
- metrics & logs
- graphql API and main database

## Hiring Project

The most significant software component in Fly.io’s architecture is our `fly-proxy`, which is implemented in Rust with Tokio. `fly-proxy` drives our Anycast network: practically all traffic for apps running on Fly.io flows through the proxy. Some of the things it does include:
- Automatically handling LetsEncrypt ALPN certificate issuance for apps and transparently terminating TLS, so apps come up the first time with a solid TLS configuration.
- Routing HTTP/1.1 and HTTP2 traffic from our edge to customer VMs, with per-customer concurrency isolation.
- Coalescing HTTP/1.1 requests received on our edge into multiplexed HTTP2 sessions to our worker servers.
- Generating fine-grained Prometheus metrics for customer applications.
- Routing WebSockets connections to VMs transparently.
- Forwarding raw TCP connections for non-web applications.
What we’d like you to do for this coding challenge is to implement your own version of this proxy, so we can see how you’d do it if you were in our shoes.

OBVIOUSLY WE ARE KIDDING.

What we want you to write is just the last bullet point from that feature list: a configurable raw TCP proxy, using the Go net package. Work will be split up into two parts.

### Part 1 - Configurable TCP proxy

First up, you're going to build a raw TCP proxy using the standard `net` Go package. Just like the real `fly-proxy`, yours will be configured with multiple apps, each with potentially many backend targets. There's already a `config` package that handles loading and watching a `config.json` file. Checkout the `config.json` file to see what needs to be supported. 

Criteria
- We can run your code with `go run`
- It listens on each port listed in the config file. Connections to any of those ports routes to one of the correct app's targets.
- If an app has multiple targets, you should have a sensible way of balancing between targets.
- The target you pick might vanish after you make the forwarding decision and before the connection completes. If there are multiple targets available, the connection should still work.

Along with your code, include a `NOTES.md` that goes over:
- a short summary of what you built
- how you add hot config reloading that doesn't break existing connections
- what might break if your proxy were under heavy load
- what, if anything, you'd improve if you had more time

### Part 2 - BPF Steering

Not long ago, `fly-proxy` would only listen on a small list of ports, just like your proxy does now. We added more ports when customers asked, but the amount of work was non-trivial, and the strict list of supported ports was annoying customers. Our solution was using an `sk_lookup` BPF program that would route TCP connections for Anycast addresses to `fly-proxy`'s listener. 

For the second part of this challenge, you'll be adding a similar BPF socket-steering program to your proxy so that any number of ports provided in your configuration can be routed to a single listener socket.

Most people we talk to haven’t done any significant BPF work, and we don’t expect you to have either. Don't worry, it's simpler than you think. Checkout the [BPF notes](BPF.md) to get started.

Criteria
- A BPF program that routes connections on any configured port to your proxy's listener.
- Your proxy still maps inbound connections to the correct app and forwards to a target.
- Your BPF code can be built with scripts or a Makefile
- Your BPF code can be configured and managed with scripts or by your Go program
- You've documented how to build and run your BPF code.

Update the `NOTES.md` file from Part 1 to cover:
- what you did to add BPF steering
- how you'd update the BPF maps when configuration changes

## What we care about

- The basic proxy needs to work.
- Even though you may be new to BPF, you're able to figure out enough to make it work.
- Your code should be clear and idiomatic.
- It's okay to deliver complicated features as written notes rather than code, but if you're able to bang them out, go for it.
- Don't spend time making this perfect. Rough edges are fine if it helps you move quickly, just explain shortcuts in the notes.
- Don't spend time on tests for this project.

## Submitting your work

You'll get an invite to a private GitHub repo in a few minutes. Do all of your work there. When you're ready for us to review, open a PR and let us know.

Let us know what email address you registered with Fly and we'll give you some credits so you can play around.

## Tips and Tricks

- You can use our test echo server for app backends: https://tcp-echo.fly.dev 
- You can use netcat to test your proxy `echo "hello" | nc -N -4 localhost 6300`

Have fun!
