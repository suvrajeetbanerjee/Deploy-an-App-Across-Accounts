# Deploy-an-App-Across-Accounts
Moving my app into another AWS Account in this multiplayer project! ✌️


* 🔗 [**NextWork Community Post**](https://community.nextwork.org/c/celebrations/just-deployed-a-docker-app-across-2-different-aws-accounts-using-ecr)
* 🔗 [**Blog Post**](https://dev.to/suvrajeet/epic-adventure-deploy-your-app-across-aws-accounts-with-docker-ecr-m4j)
* 🔗 [**Documentation**](https://mega.nz/file/f65kDLoa#qbhIYhMshUb8i5NC5ViAAoR9mbQvWfy26Kz3VrWyIGs)
* 🔗 [**X-Post**](https://x.com/_suvrajeet_/status/1954960494013608002) 


## 🏷️ Skills & Technologies

<!-- Badge row (shields.io, style=for-the-badge to match your uploaded look) -->
![Docker](https://img.shields.io/badge/Docker-Containerization-blue?style=for-the-badge&logo=docker&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-Cloud-orange?style=for-the-badge&logo=amazon-aws&logoColor=white)
![ECR](https://img.shields.io/badge/ECR-Registry-ff8c00?style=for-the-badge)
![IAM](https://img.shields.io/badge/IAM-Access--Control-7f8cff?style=for-the-badge)
![Repo Policies](https://img.shields.io/badge/Repo%20Policies-Permissions-6c757d?style=for-the-badge)
![Cross-Account](https://img.shields.io/badge/Cross--Account-Sharing-2ea44f?style=for-the-badge)
![AWS-CLI](https://img.shields.io/badge/AWS_CLI-Automation-238636?style=for-the-badge)
![Image Tagging](https://img.shields.io/badge/Image%20Tagging-Versioning-0a0?style=for-the-badge)
![Troubleshooting](https://img.shields.io/badge/Troubleshooting-403%20Fixes-red?style=for-the-badge)
![Debugging](https://img.shields.io/badge/Debugging-CloudTrail%20/%20STS-ff66a3?style=for-the-badge)
![Dockerfile](https://img.shields.io/badge/Dockerfile-Best%20Practices-007ec6?style=for-the-badge)
![Nginx](https://img.shields.io/badge/Nginx-Static%20Hosting-66cc33?style=for-the-badge&logo=nginx&logoColor=white)
![Automation](https://img.shields.io/badge/Automation-Scripts%20%26%20CI-0052cc?style=for-the-badge)
![GitHub%20Actions](https://img.shields.io/badge/GitHub_Actions-CI%2FCD-2088ff?style=for-the-badge&logo=githubactions&logoColor=white)
![Collaboration](https://img.shields.io/badge/Collaboration-Shared%20Artifacts-6f42c1?style=for-the-badge)
![ECS%20/%20Fargate](https://img.shields.io/badge/ECS%20%2F%20Fargate-Next%20Step-4a4a4a?style=for-the-badge)
![Assume%20Role](https://img.shields.io/badge/AssumeRole-Secure%20Access-0070ff?style=for-the-badge)
![Access%20Control](https://img.shields.io/badge/Access%20Control-Least%20Privilege-f9b115?style=for-the-badge)





# 🚀 Epic Adventure — Deploying My App Across AWS Accounts with Docker & ECR

**Short version:** I actually did this. I containerized a tiny app, pushed it to Amazon ECR, granted cross-account access, and pulled the image from another AWS account — all while learning the real-life gotchas most tutorials skip. This repo documents the exact steps I took, what broke, what I learned, and the skills I walked away with. ✨ ([DEV Community][1])

---

## 📌 Why this repo (TL;DR)

I built this to prove one thing: **cross-account ECR workflows are straightforward when you understand auth, repo policies, and image tagging.** I share my commands, the mistakes I made, how I fixed them, and the skills I actually leveled up. For background reading that inspired this exercise, see my write-up and the ECR multi-account patterns. ([DEV Community][1], [Amazon Web Services, Inc.][2])

---

## 🧾 Repo Snapshot

```
.
├── app/
│   ├── index.html
│   └── Dockerfile
├── scripts/
│   ├── ecr-login.sh
│   ├── build-tag-push.sh
│   └── repo-policy-sample.json
├── notes/
│   └── runbook.md        # first-person playbook + troubleshooting log
└── README.md
```

---

## ✅ What I did

1. Built a tiny static site and a minimal `Dockerfile` using `nginx:alpine`.
2. Created an ECR repo in **Account A**, configured AWS CLI for my IAM user, and authenticated Docker to ECR.
3. Built, tagged, and pushed the image to Account A’s ECR.
4. Hit **403 Forbidden** when my friend (Account B) tried to pull — I fixed it by adding a repo policy that grants Account B explicit `BatchGetImage` / `GetDownloadUrlForLayer` permissions.
5. Re-authenticated from Account B and successfully pulled & ran the image locally. Victory lap. 🥳 ([DEV Community][1])

---

## 🧠 Commands I ran (copy-paste friendly)

### Authenticate Docker to ECR (one-liner)

```bash
aws ecr get-login-password --region <region> \
  | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.<region>.amazonaws.com
```

### Build, tag, push

```bash
docker build -t cross-account-app:latest ./app
docker tag cross-account-app:latest \
  <acct-a-id>.dkr.ecr.<region>.amazonaws.com/my-app:latest
docker push <acct-a-id>.dkr.ecr.<region>.amazonaws.com/my-app:latest
```

### Pull from buddy account (after they added permissions)

```bash
aws ecr get-login-password --region <region> \
  | docker login --username AWS --password-stdin <acct-a-id>.dkr.ecr.<region>.amazonaws.com

docker pull <acct-a-id>.dkr.ecr.<region>.amazonaws.com/my-app:latest
```

---

## 🔐 Repo policy I used to fix the 403s

I added this policy on the ECR repo in **Account A** to allow Account B to pull images:

```json
{
  "Version": "2008-10-17",
  "Statement": [
    {
      "Sid": "CrossAccountPull",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::<acct-b-id>:root" },
      "Action": [
        "ecr:BatchGetImage",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetRepositoryPolicy",
        "ecr:DescribeRepositories",
        "ecr:ListImages"
      ]
    }
  ]
}
```

That single policy cleared the 403s for me. If you want to scope it tighter, point `Principal` to a specific IAM role or user ARN instead. (I tried both during the exercise.) ([Amazon Web Services, Inc.][2])

---

## 📚 What went wrong & how I fixed it

* **Login said “Login succeeded” but pulls still failed** — reason: login to registry succeeded, but the repo policy didn’t permit cross-account pulls. Fix: add the repo policy above.
* **Tag mismatch** — I pushed `v1` and my buddy tried pulling `latest`. Always agree on tags. I now push both `v1` and `latest` during demos.
* **Confusing IAM errors** — debug process: check CloudTrail + ECR repo policy, then test with the exact `docker pull` and `aws sts get-caller-identity` to confirm which principal is used.

---

## 🏆 Achievements — what I actually learned

* I moved from *theory* → *applied practice* for ECR cross-account sharing: I resolved real permission issues and documented the fixes. ([DEV Community][1])
* I can authenticate Docker to ECR reliably, tag images correctly for account-specific registries, and explain why a repo policy is necessary.
* I learned to debug 403s by checking (a) repo policy, (b) IAM permissions, and (c) the exact principal used for the pull request.
* I automated the login → build → push flow into a shell script so I don’t repeat manual typing mistakes anymore.

---

## 🛠 Skills I gained

* Docker: image build, tag, and local run
* AWS CLI: `aws ecr` flows, authentication, and quick debugging
* AWS ECR: repo permissions, policies, and cross-account architecture concepts
* Troubleshooting: reading permissions errors, using `aws sts get-caller-identity`, and iterating quickly
* Collaboration ops: getting two accounts to share private artifacts securely — a real-world teamwork skill

---

## 💡 Learning outcomes

* I can explain why ECR is private by default and why a repo policy is the right tool for cross-account pulls. ([Amazon Web Services, Inc.][2])
* I can get a container from **my laptop** into **Account A → Account B** with minimal steps and documented commands. ([DEV Community][1])
* I can show this end-to-end flow as a demo in under 20 minutes now that I’ve automated the repetitive parts.

---
<!--
## 📷 Visuals & proof

Images in this README are proxied by GitHub’s image proxy (Camo), which is why the top images are `camo.githubusercontent.com` URLs. Camo ensures images load over HTTPS and hides client details — neat for shared READMEs. ([GitHub Docs][3], [GitHub][4]) -->


## ✅ Next steps / where I’ll take this

* Add GitHub Actions workflow to build and push on tag — so images are produced by CI.
* Extend the demo to a simple ECS/Fargate task in Account B pulling from Account A’s ECR.
* Add automated, minimal-privilege IAM role assumption for sharing (no root ARNs).
  If you want any of these in this repo, tell me which one and I’ll add the first cut.

---

## 📎 References & further reading

* My narrative + playbook (blog): *Epic Adventure — Deploy Your App Across AWS Accounts with Docker & ECR*. ([DEV Community][1])
* AWS guidance on multi-account ECR patterns. ([Amazon Web Services, Inc.][2])
<!--
* Why GitHub proxies images via Camo (image proxy explanation). ([GitHub Docs][3], [GitHub][4])
-->


---

## ✍️ Final thoughts

This wasn’t just a tutorial copy/paste — I actually did the steps, fixed the 403s, automated what annoyed me, and wrote down the exact commands that worked. If you clone this repo and follow `notes/runbook.md`, you’ll hit the same milestones I did. Old-school persistence + modern tooling = results. Let’s keep it practical and ship things that work. 💪


[1]: https://dev.to/suvrajeet/epic-adventure-deploy-your-app-across-aws-accounts-with-docker-ecr-m4j?utm_source=chatgpt.com "Deploy Your App Across AWS Accounts with Docker & ECR!"
[2]: https://aws.amazon.com/blogs/containers/amazon-ecr-in-multi-account-and-multi-region-architectures/?utm_source=chatgpt.com "Amazon ECR in Multi-Account and Multi-Region Architectures - AWS"

---

## 🙏 Acknowledgments
<!--
Shoutout to the amazing online resources and communities that guided me through this. Couldn’t have done it without you!
-->

This learning journey was powered & supported by NextWork's structured approach to cloud education, which made breaking down complex concepts into digestable-byte-sized-hands-on practical accessible through systematic skill building & clear-actionable steps.

This write-up is based on - NextWork's ⏩ [Deploy an App with Docker Challenge](https://learn.nextwork.org/projects/aws-compute-eb)! Give it a try 😃😎 !!!



  
<!-- **Repo Readme** ⏩  *Comming very Soon!* -->

<!-- Tomorrow dated 11.08.25* -->
<!-- 
  * [Documentation](mega.link) – Documentations Name -->
