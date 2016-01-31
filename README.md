(add intro here)

## Set up an IAM User

First we need to set up a user that we will use specifically for backups, which has the permissions we need (and just these permissions).  To do this:
1. Go to "Identity and Access Management" section in the AWS console.
2. Click on "Users" on the left.
3. Enter one user name, "snapshot-backup".  Click Create.
4. Download Credentials and Close
5. Click on the new user and then click on Inline Policies
6. Create a new policy, and click on Custom Policy
7. Name the custom policy "snapshot-backup" and use the below for the policy document.  Then click Apply Policy.

Policy to use:
```
{
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateSnapshot",
                "ec2:CreateTags",
                "ec2:DeleteSnapshot",
                "ec2:DescribeSnapshots",
                "ec2:DescribeTags"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

## Gather EBS info

(get volume ID and regions)

## Set up the AWS CLI

On the instance where you want to run the snapshot tool, first make sure you have the AWS CLI installed.  On Ubuntu, you can run:
```
sudo apt-get install awscli
 ```
 
Then we need to configure a "ssbackup" profile that will be used in the backup script.
```
aws configure --profile=ssbackup
AWS Access Key ID [None]: (enter the id found in the credentials file you downloaded above)
AWS Secret Access Key [None]: (enter the key found in the credentials file you downloaded above)
Default region name [None]: (enter the region where your EBS drive is located)
Default output format [None]: json
```

## Download and test the script

Now we need download the script that will be used for backups:
```
sudo curl -o /usr/local/bin/ssbackup.py https://raw.githubusercontent.com/bdeitte/ghost-on-aws/master/scripts/ssbackup.py
less /usr/local/bin/ssbackup.py
```
The second command is there because it's always good practic to really look through what you've downloaded and as a sanity check that it's there.

python3 /usr/local/bin/ssbackup.py --volume-ids={vol} --expiry-days=7

## Things to do

(info on monitoring and using different auth)

## Credits

I've modified the script and steps from a [https://www.flynsarmy.com/2015/06/how-to-schedule-daily-rolling-ebs-snapshots/](blog post on the subject).  Thank this person and not me for this project, as they are largely responsible for it.
