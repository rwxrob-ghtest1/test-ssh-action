# Sample GitHub action with remote deployment

We assume everything has been built and/or released by other workflow actions. There is just conventional agreement on how stuff it retrieved and pushed to the remote target for deployment.

## Caveats

* GitHub Action secrets and variables must be set by validating human admin (not automated)
* NEVER disable strict hostkey checking!
* Always use low-privledged user on target host to receive the incoming files (never root)
* Content and permissions of `deploy` are outside scope of this task

## Deploy workflow

1. Read the external source URL list from `jobs.<jobs_id>.outputs`
1. Infer the checksum file from `jobs.<jobs_id>.outputs.checksum_url`
1. Copy from (one or more) external source into `dist`
1. Copy `deploy` script into `dist`
1. Write `ssh.key` from `secrets.SSH_PRIVATE_KEY` with proper perms
1. Update hostkey with `ssh-keyscan` for specific target host from `vars.HOSTKEY`
1. Assign random UUID to `DEPLOY_UUID` env variable (ensure *actually* unique)
1. Tar/compress `dist` into `${{ vars.PREFIX }}-DEPLOY_UUID.tgz`
1. Create `checksum` for file validation
1. Set timeout trap/alarm by which to cancel transfer if not complete
1. Copy `tgz` file to destination with `scp -i ssh.key`
1. Remotely validate checksum to ensure proper transfer
1. Remotely execute `deploy` contained in top level of target directory (with `DEPLOY_UUID`)

