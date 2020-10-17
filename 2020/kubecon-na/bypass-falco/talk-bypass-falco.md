theme: Work, 1
autoscale: true
build-lists: true
slidenumbers: true
slidecount: true
slide-transition: fade(0.3)
footer: [@leodido](https://twitter.com/leodido)



# Bypass
# [fit] [Falco](https://github.com/falcosecurity/falco)

![original](assets/img/kubecon2020na-circle-bg.png)

<br>

## Leonardo Di Donato - 20 Nov 2020

[.hide-footer: true]
[.slidenumbers: false]

---

![left filtered](assets/img/leo-at-rejects-sandiego-half.png)

# Whoami

<br>
## Leonardo Di Donato

## Open Source Software Engineer
## Falco Maintainer

<br>
## @leodido [![inline fit](assets/img/twitter.png)](https://twitter.com/leodido) [![inline fit](assets/img/github.png)](https://github.com/leodido)

![inline fit](assets/img/cncf-incubating-white.png) ![inline fit](assets/img/falco-white.png) ![inline fit](assets/img/sysdig-white.png)

[.hide-footer: true]
[.slidenumbers: false]

---

## A timeline always works fine

![inline](assets/img/falco-timeline.svg)

---

![left](assets/img/bg1.jpg)
# Contents

* Rationale
* Falco
   * What's runtime security?
   * How does it work?
* Bypass!
   * /honk

[.hide-footer: true]
[.slidenumbers: false]

---

![fit](assets/img/bg2.jpg)

> You gonna get fired for this.
> It's a mistake.
-- my father.

---

![original filtered](assets/img/bg3.jpg)

> other quote
-- other author.

---

### Security

# [fit] Prevention + Detection

[.column]
Use **policies** to _change the behavior_ of a process by preventing syscalls from succeeding (also killing the process).

![inline](assets/img/selinux.svg) ![inline](assets/img/apparmor.svg) ![inline](assets/img/kubernetes.png)

[.column]
Use **policies**  to _monitor the behavior_ of a process and notify when its behavior steps outside the policy.

![inline](assets/img/falco-logo-only.png)

[.autoscale: false]

---

# [fit] Prevention is not enough.
#### Combine with runtime detection tools. Use a [defense-in-depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing) :link: strategy.

![inline fit](assets/img/cloudnativearch.svg)

---

![right filtered](assets/img/kelly.jpg)

# [fit] Runtime Security

She’s **Kelly**. :broken_heart:

I have a lock on my front door and an alarm. She alerts me when things aren’t going right, when little bro is misbehaving or if there’s someone suspicious outside or nearby.

She detects **runtime anomalies** in my life at home.

**Still...** Bad people were able to defy her and break into my house.

---

<br>
<br>
# [fit] There is no such thing
# [fit] as perfect security.

---

# ...

---

# ...

---

# Categories

---

# Override binaries


```yaml
rule: ...
```

---

## Demo

...

---

# Mitigation

---

# Missing renameat2

---

# Missing excecveat

---

# Mitigations

---



---

# ...

---

![](assets/img/bg4.jpg)

# [fit] Thanks and Honks!
### [fit] Does anyone have any questions?

![inline 60%](assets/img/kubecon2020na.png)

[.column]
* [twitter.com/leodido](https://twitter.com/leodido)
* [gh:leodido](https://github.com/leodido)
* [gh:falcosecurity/falco](https://github.com/falcosecurity/falco)
* [slack.k8s.io](https://slack.k8s.io), #falco channel

[.column]
![inline](assets/img/handwaving.png)

[.build-lists: false]
[.hide-footer: true]
[.slidenumbers: false]