# packer-ansible-docker-gitlab

Packer template which builds a Docker Compose based GitLab AMI. GitLab will be
running on ports 80, 443, and 22 by default. Instance SSH access is moved to port 2222.
It should launch directly to a running instance of GitLab, if port 80 doesn't come
up after a couple of minutes then something is wrong.

## Configuration

Everything application-related is configured in `playbook.yml`'s vars, except the
new system SSH port, which is hardcoded to 2222. The AMI-related configs are all in
`packer.json`.

You'll probably want to change these:
* `playbook.yml`:
  * `gitlab_omnibus_config` - Multi-line string which contains `gitlab.rb` configs.
  * `users` - List of dicts whose keys directly configure [inhumantsar.local-user](https://github.com/inhumantsar/ansible-role-local-user).
  * `service_env` - Arbitrary string to ID environment. eg: dev, test, prod
* `packer.json`:
  * `region` - AWS region ID
  * `source_ami` - Update to latest in the appropriate version. [More info](https://wiki.centos.org/Cloud/AWS)
  * `instance_type` - If this is changed to the t2 family, you must remove the spot market configs.
  * `run_tags` - Tags to apply to instances launched with the AMI.
  * `launch_block_device_mappings` - May want to increase disk size or change disk type.

### AWS Credentials

These need to be configured ahead of time and be available to Packer in the "usual" places
for AWS. For most people, this means `~/.aws/credentials`. If that file or that directory
doesn't exist, then see _[Configuring the AWS CLI](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)_
to get started.

## Build

`packer build packer.json`

Or, with Docker:
```bash
docker run --rm \
  -v $(pwd):/code -w /code \
  -v $HOME/.aws:/root/.aws \
  hashicorp/packer:light build packer.json
```
