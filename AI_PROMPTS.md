# Generative AI Disclosure

Using AI tools for this challenge is allowed. Be honest here — an accurate
"I used AI heavily" is worth more to us than an inaccurate "I did not".

**Did you use AI tools?** (yes / no): yes

**Which ones?** (e.g. ChatGPT, Claude, Copilot, Cursor): Gemini

> If you answered **no**, write "N/A" in sections 1 and 2 and skip to
> section 3. That is a perfectly good answer.

---

## 1. Key prompts

List the main prompts you used. Paraphrasing is fine.

1. `how to run nginx apt install from dockerfile without overriding existing nginx conf on container`
2. `how to configure network on docker compose so that containers will be able to communicate`
3. `how to configure health checks in docker compose for url checkup`
4. `how to initialize container only after other container is healthy in docker compose`
5. `permission error in github workflow when attempting to execute file-reasons and how to fix`

bonus:
6. `what lightweight base image should i use for building minimal weight nginx`
7. `what non root user can i use to run nginx with`
8. `how to configure a private internal network so that containers are inaccessible from outside`
9. `best practice nginx configuartion`
10. `how to configure ssl certs for nginx container`



---

## 2. Where the AI got it wrong

Describe **one concrete case** where the AI's answer was wrong, outdated, or
sub-optimal. Cover: what it suggested, how you noticed it was wrong, and what
you did instead. 3–5 sentences.

when asked to configure health-check the ai-offered a solution that just checks the wget command success - without examining the payload - but the curl/wget action may succeed also on an unhealthy app. therefore The health check might get a false positive.
I fixed it by validating against the body of the response

---

## 3. Anything in your submission you could not fully explain?

We would rather know. Write "nothing" if that is the case.

- meaning of some of the configurations in the nginx conf file
