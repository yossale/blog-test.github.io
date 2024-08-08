---
title: Port forwarding on ec2
---
we want to get to `prod1`
but it's only accessible through `mid`
and we want to reach it from `localhost`

So:

```
ssh -fN -L<LocalPort>:<prod1 ip/hostname>:<prod port> my-keypair.pem ec2-user@<mid ip>
```

or

```
ssh -fN -L<LocalPort>:<Where you want to get>:<port> my-keypair.pem ec2-user@<through where>
```

and eventually what we get is this: we want every request sent to localhost:1234 to go to 1.2.3.4:5678 through 4.3.2.1

```
ssh -fN -L1234:1.2.3.4:5678 my-keypair.pem ec2-user@4.3.2.1
```




