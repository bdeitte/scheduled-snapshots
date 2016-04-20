A step-by-step guide for taking daily, rotating snapshots of EBS volumes.  You will get one new snapshot every day, and any snapshot over 7 days old will be deleted automatically.

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
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": "ec2:CreateSnapshot",
            "Effect": "Allow",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "ec2:CreateTags",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "ec2:DeleteSnapshot",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "ec2:DescribeSnapshots",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "ec2:DescribeTags",
            "Resource": "*"
        }

    ]
}
```

## Gather EBS info

In the AWS console, go to EC2 and Volumes.  Copy down the Volume ID for any drive that you want to take backups for.  Also note the region you are in, shown in the upper-right of the screen.  Write down the short name for the region, referred to as the Code [in the documentation](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html).

## Set up the AWS CLI

On the instance where you want to run the snapshot tool, first make sure you have the AWS CLI installed.  On Ubuntu, you can run:
```
sudo apt-get install awscli
 ```

Then we need to configure the tool to use the "ssbackup" profile that will be used in the backup script.
```
aws configure --profile=ssbackup
AWS Access Key ID [None]: (enter the id found in the credentials file you downloaded above)
AWS Secret Access Key [None]: (enter the key found in the credentials file you downloaded above)
Default region name [None]: (enter the Code for region that was written down above)
Default output format [None]: json
```

## Download and test the script

Now we need download the script that will be used for backups:
```
sudo curl -o /usr/local/bin/ssbackup.py https://raw.githubusercontent.com/bdeitte/scheduled-snapshots/master/scripts/ssbackup.py
less /usr/local/bin/ssbackup.py
```
The second command is there because it's always good practice to really look through what you've downloaded and as a sanity check that it's there.

Now you can test out the script.  Substitute in the volume IDs that you wrote down.  You can enter multiple volume IDs separated by commas.
```
sudo python3 /usr/local/bin/ssbackup.py --volume-ids={vol},{vol} --expiry-days=7
```
If there is nothing but short processing output, then things are working.  You should see the snapshot show up in the AWS UI shortly, in EC2 under Snapshots.

## Setting up cron

Now we need to set up the script to run every day automatically.  Use cron for this:
```
sudo crontab -e
```
Create a new line at the bottom of the file that appears.  It should contain the same volume IDs that you tested with above, but the line itself is a bit different.
```
20 0 * * * python3 /usr/local/bin/ssbackup.py --volume-ids={vol} --expiry-days=7
```
You can change the first two numbers in that line if you would like it to run at a slightly different time.  This is set up to run at 12:20 am.  If you would like it to run at 2:00am, for instance, you would change "20 0" to "0 2".

Save the file, and you're all set!  The script will now run every day, and you'll always have 7 days of snapshots to use in case of an emergency.

## Things still to do in this guide (someday)

- Add more troubleshooting info in here if things don't get set up right or snapshots aren't showing up.
- Use an IAM instance role rather than the ID and key to set up the CLI
- Alert on errors when running the snapshot script or when cron isn't running.  I've used DataDog for this kind of thing in the past, but there's lots of others free or cheap tools that could be used too.

## Credits

I've modified the script and steps from a [blog post on the subject](https://www.flynsarmy.com/2015/06/how-to-schedule-daily-rolling-ebs-snapshots/).  Thank this person and not me for this project, as they are largely responsible for it!
