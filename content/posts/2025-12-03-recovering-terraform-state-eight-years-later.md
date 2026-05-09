---
layout: post
title: Recovering Terraform State, Eight Years Later
date: 2025-12-03
tags: ["terraform"]
---

Back in 2017 I wrote about [recovering orphaned Terraform state](/posts/2017-04-26-recovering-terraform-state/) with the `import` command. That post is still up because the situation is still the same: somebody created infra without managing state, somebody else has to bring it back under control, and you're the somebody else. What's changed is the tooling. Three things, mostly.<!--more-->

I'm not going to re-explain what state is or why you'd want to recover it. The 2017 post does that. This is the "what would I do today" version.

## Import blocks instead of `terraform import`

The CLI command still works. I'd stop reaching for it.

Since 1.5 (June 2023) Terraform supports `import` blocks in configuration:

```hcl
import {
  to = aws_instance.example
  id = "i-0e3f3db1d2c5a4520"
}
```

Three things this gets you that the CLI didn't. It's plannable, so you can review what's about to happen before state is touched. It's reviewable, because the import lives in a PR like everything else. And it bulks: drop a `for_each` on the import block and recover a list of IDs in one apply.

```hcl
locals {
  instance_ids = [
    "i-03676fa6ba43fbb9f",
    "i-09f51a313146856cd",
  ]
}

import {
  for_each = toset(local.instance_ids)
  to       = aws_instance.web[each.key]
  id       = each.value
}
```

If you've also lost the configuration (which is half the point of a recovery), `terraform plan -generate-config-out=generated.tf` will write resource blocks for you, populated with the actual attribute values from AWS. The output is verbose; it includes computed attributes, defaults, everything. You'll trim it. But it gets you to "I have a configuration that matches the live resource" without copying values out of the console by hand, which in 2017 was the whole tedious part.

After the apply, the import blocks have done their job. Delete them.

## `moved` and `removed` blocks instead of `state mv` / `state rm`

The state-manipulation CLI commands also still work, and I also don't use them.

`moved` blocks (1.1+, late 2021) cover the rename, reorganize, and convert-count-to-for_each cases. `removed` blocks (1.7+, early 2024) cover "stop managing this without destroying it":

```hcl
moved {
  from = aws_instance.web
  to   = module.compute.aws_instance.web
}

removed {
  from = aws_instance.legacy_server
  lifecycle {
    destroy = false
  }
}
```

Same argument as above. State changes that used to live in shell history now live in version control. That's the whole point.

## S3 native locking instead of DynamoDB

This one's the structural shift. The 2017 post assumed S3 plus a DynamoDB table for locking, and that's been the default for so long it's reflex. As of Terraform 1.10 (November 2024), the S3 backend has its own locking via `use_lockfile = true`, and the DynamoDB table is no longer needed:

```hcl
terraform {
  backend "s3" {
    bucket       = "my-state"
    key          = "infra/terraform.tfstate"
    region       = "us-east-1"
    encrypt      = true
    use_lockfile = true
  }
}
```

If you're standing up a new project today, skip the DynamoDB table. If you have one, it'll keep working for now (the DynamoDB locking path is formally deprecated and will be removed in a future minor release), but there's no reason to provision a new one. One less moving part, one less IAM policy to maintain, one less thing to forget to clean up.

## What I'd actually do

If I lost state on a small configuration tomorrow, I'd write `import` blocks for each resource by hand and run `terraform plan -generate-config-out=…`, then clean up the generated file. If I lost state on something bigger, I'd write the `import` blocks with `for_each` keyed off resource lists I pulled from the AWS API, and trust generate-config-out for the configuration. Either way, the workflow now is "edit configuration, plan, apply" instead of "run a sequence of CLI commands while sweating." The 2017 version of me would have been delighted.
