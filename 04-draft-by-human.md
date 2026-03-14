Subject: direct usage of the official images is no longer supported

Hi,
I found the following notice in the OpenTofu documentation https://opentofu.org/docs/intro/install/docker/ :

> Starting with OpenTofu 1.10, direct usage of the official images is no longer supported.

This means that we can no longer run opentofu with `docker run ghcr.io/opentofu/opentofu`, right?

This is really inconvenient for the users of OpenTofu.
Providing running-ready official images is a great advantage for opensource projects.
I'm afraid that this makes it hard to introduce opentofu for many users, and it might make opentofu less popular.


The trigger of this change seems to be this issue https://github.com/opentofu/opentofu/issues/1931, and I suppose why OpenTofu no longer support direct usage are followings:

* The base image (alpine) is not updated frequently, and it's difficult to update the base image.
* That might cause security issues.
* That might cause outage of CA certificates.

"Not supported" is a too strong word to users.
Opentofu is open source project, provides no warranty, and is "not supported" in the beginning. Explicitly stating the "not supported" looks too restrictive to me.

How about handling this issue in this way?:

* Semi-automatic updating of the alpine version by dependabot
    * https://docs.github.com/en/code-security/reference/supply-chain-security/supported-ecosystems-and-repositories#docker
    * Or, just use `latest` tag is another option.
    * We might need to add a test code to check the image runs correctly. Not so complicated tests, just need running `tofu --version`.
* Not "Not supported" but "Limitation":
    * The version of the base image is not always up-to-date. Recommended to create your own image especially in security-sensitive environments.
    * The base image can be changed in the future versions.


I know maintaining the project requires much resources and this ask for too much to core teams.
But I believe this results much more benefits for the users of opentofu.
Please consider this request.
