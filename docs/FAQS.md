## FAQS

Here's a compilation of Frequently Asked Questions (FAQs). Feel free to add to our FAQs by raising a Pull Request (PR).

---

## Commands

### My command isn't recognised and I got the following error, how do I get it to work?

- When copying and pasting commands from Github viewed on a browser, the formatting can result in the command not being recognised in your terminal.

- We recommend you try to copy and paste the command from your code editor instead e.g. Visual Studio Code, Notepad++ etc.

- While you're learning, we encourage you to type commands out by hand so that you can reinforce your knowledge a bit better.

---

## Terraform

### Terraform apply produced an error, what can I try next?

- If you get an error during `terraform apply`, let's determine if it's just a one-off issue or does the command fail consistently in the same place i.e. can you reproduce the error messages?

- Run `terraform apply` again to see what outcome you get.

- You can also try to run `terraform destroy` first and then run `terraform apply` again.

---

### How do I remove Terraform's state to start afresh again?

- When you want a blank slate to work from, it is advisable to ensure that the state has been fully cleared down.

- Run `terraform destroy`

- Once that's complete run:

```bash
rm -rf .terraform
```
- And then run:

```bash
rm -rf terraform.tfstate*
```

---

### Where do I run the `terraform destroy` command?

- You need to run the `terraform destroy` command from the location where your terraform config is