Thanks for the detailed reply.

What I take from your explanation is that running OpenTofu in Docker in a useful way tends to require substantial Docker expertise.

I agree that running applications in containers often demands both solid Docker knowledge and a good understanding of how the application itself expects to interact with its surroundings (credentials, other software, and so on, as you described).

That trade-off makes a simple `docker run`-style workflow less appealing for me, so I'm likely to run OpenTofu outside Docker. I need to use different OpenTofu versions across different projects, so for now I'm evaluating `mise` as one option.

This explanation addresses my original concern, so I'm going to close this issue. Thanks again for the thoughtful write-up.
