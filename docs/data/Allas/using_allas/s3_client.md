
# The S3 client

This chapter describes how to use the Allas object storage service with the **s3cmd** command line client. This client uses
the _S3_ protocol that differs from the _Swift_ protocol used in the [rclone](./rclone.md), [swift](./swift_client.md) and [a-commands](./a_commands.md) examples. 
Thus, data that has been uploaded to Allas using these tools should not be downloaded with s3cmd and vice versa.

From the user perspective, one of the main differences between s3cmd and Swift based tools is that rclone, swift and a-tools connections remain valid for eight hours 
at a time but with s3cmd, the connection remains open permanently. The permanent connection is practical in many ways but it has a security aspect: 
if your CSC account is compromised, so is the object storage space.

The syntax of the `s3cmd` command:
```text
s3cmd -options command parameters
```

The most commonly used _s3cmd_ commands are:

| s3cmd command | Function |
| :---- | :---- |
| mb | Create a bucket |
| put | Upload an object |
| ls | List objects and buckets |
| get | Download objects and buckets |
| cp | Move object |
| del | Remove objects or bucket |
| md5sum | Get the checksum |
| info | View metadata |
| signurl | Create a temporary URL |
| put -P | Make an object public |
| setacl --acl-grant | Manage access rights |


The table above lists only the most essential s3cmd commands. For more complete list, visit the [s3cmd manual page](https://s3tools.org/usage) or type:
```text
s3cmd -h
```
If you use Allas on Puhti or Taito, all required packages and software are already installed, and you can jump to the section [s3cmd with supercomputers](#s3cmd-with-supercomputers). You can skip the installation chapter *Getting started with s3cmd* below.

## Getting started with s3cmd

To configure s3cmd connection, you need have OpenStack and s3cmd installewd in your machine. 

**OpenStack s3cmd installation:**

Fedora/RHEL derivatives:
```text
sudo yum update
sudo yum install python3
sudo pip3 install python-openstackclient
sudo yum install s3cmd
```
Debian derivatives:
```text
sudo apt-get update
sudo apt install python-pip python-dev
sudo pip install python-openstackclient
sudo apt install s3cmd
```
OSX:
```text
python3 virtualenv
pip3 install s3cmd
s3cmd
```

Please refer to [http://s3tools.org/download](http://s3tools.org/download) and [http://s3tools.org/usage](http://s3tools.org/usage) for upstream documentation.

Once you have OpenStack and s3cmd instralled in your environment, you can download the [allas_conf](https://raw.githubusercontent.com/CSCfi/allas-cli-utils/master/allas_conf)
script to set up S3 connect to your allas project. 
```text
wget https://raw.githubusercontent.com/CSCfi/allas-cli-utils/master/allas_conf
source allas_conf --mode s3cmd --user your-csc-username
```
Note that you shoud use the `--user` to  define your CSC-username for the command. The configuration command first asks your 
CSC password and then asks you to define the Allas project. After that the tool creates a key file for S3 conection to and stores it to the default location.


## s3cmd with supercomputers

To use s3cmd in Taito or Puhti you must first confugure the connection with commands:
```text
module load allas
allas-conf --mode s3cmd
```

The configuration process asks first your CSC password. Then it lists your Allas projects and asks you to define the name of the project to be used. The configuration 
information is stored in the file _$HOME/.s3cfg_. This configuration needs to be defined only once. In the future, s3cmd will use the object storage connection 
described in the _.s3cfg_ file automatically. However, if you wish to change the Allas project that s3cmd uses, you only need to run the configuration command again.

## Create buckets and upload objects

You can create a new bucket with command:

```text
s3cmd mb s3://my_bucket
```

Upload a file to a bucket:
```text
s3cmd put my_file s3://my_bucket
```

## List objects and buckets

List all buckets in a project:
```text
s3cmd ls
```

List all objects in a bucket:
```text
s3cmd ls s3://my_bucket
```

Display information about a bucket:
```text
s3cmd info s3://my_bucket
```

Display information about an object:
```text
s3cmd info s3://my_bucket/my_file
```

## Download objects and buckets

Download an object:
```text
s3cmd get s3://my_bucket/my_file new_file_name
```
The parameter *new_file_name* is optional. It defines a new name for the downloaded file.

Using the command `md5sum`, you can check that the file has not been changed or corrupted:
<pre>
$ <b>md5sum my_file new_file_name</b>
   39bcb6992e461b269b95b3bda303addf  my_file
   39bcb6992e461b269b95b3bda303addf  new_file_name
</pre>
In the above example, the checksums match between the original and the downloaded file.

Download an entire bucket:
```text
s3cmd get -r s3://my_bucket/
```

## Move objects

Copy an object to another bucket:
```text
s3cmd cp s3://sourcebucket/objectname s3://destinationbucket
```

For example:
<pre>
$ <b>s3cmd cp s3://bigbucket/bigfish s3://my-new-bucket</b>
remote copy: 's3://bigbucket/bigfish' -> 's3://my-new-bucket/bigfish'
</pre>

Rename the file while copying it:
<pre>
$ <b>s3cmd cp s3://bigbucket/bigfish s3://my-new-bucket/newname</b>
remote copy: 's3://bigbucket/bigfish' -> 's3://my-new-bucket/newname'
</pre>

## Delete objects and buckets

Delete an object:
```text
s3cmd del s3://my_bucket/my_file
```

Delete a bucket:
```text
s3cmd rb s3://my_bucket
```
**Note:** You can only delete empty buckets.

## s3cmd and public objects

In this example, the object _salmon.jpg_ in the pseudo folder _fishes_ is made public:
<pre>
$ <b>s3cmd put fishes/salmon.jpg s3://my_fishbucket/fishes/salmon.jpg -P</b>
Public URL of the object is: http://a3s.fi/my_fishbucket/fishes/salmon.jpg
</pre>

**Note:** The above client outputs a URL with `http://` (which is not open in the object storage firewall). The URL needs to be manually changed to `https` if this kind of a client is used.

## Giving another project read access to a bucket

You can control the access rights using the command ```s3cmd setacl ```. This command requires the UUID (_universally unique identifier_) of the project you want to grant access to. The ID can be found in <a href="https://pouta.csc.fi/dashboard/identity/" target="_blank">https://pouta.csc.fi/dashboard/identity/</a> or using the command ```openstack project show $project_name ```. You need access (membership) to the project to find out the UUID.
 
In the Pouta web UI, you can see only buckets that the members of your project have created. If your project has been granted project read access to a bucket with the s3cmd client: 
 
 * The members of your project can list and fetch files with _python-swiftclient_.
 * _swift list_ does <u>not</u> display the bucket.
 * _s3cmd ls_ displays the bucket.
 
Grant read access:
```text
s3cmd setacl --acl-grant=read:$other_project_uuid s3://my_fishbucket
```

View permissions:
<pre>
$ <b>s3cmd info s3://my_fishbucket|grep -i acl</b>
   ACL:       other_project_uuid: READ
   ACL:       my_project_uuid: FULL_CONTROL
</pre>

Revoke read access:
```text
s3cmd setacl --acl-revoke=read:$other_project_uuid s3://my_fishbucket
```

## Use example

In this example, we store a simple dataset in Allas using s3cmd.

First, create a new bucket. The command `s3cmd ls` reveals that the object storage is empty at first. Then, use the command `s3cmd mb` to create a new bucket called _fish-bucket_.

<pre>
$ <b>s3cmd ls</b>
ls
 
$ <b>s3cmd mb s3://fish-bucket</b>
mb s3://fish-bucket/
Bucket 's3://fish-bucket/' created

$ <b>s3cmd ls</b>
ls
2018-03-12 13:01  s3://fish-bucket
</pre>
It is recommended to collect the data to be stored as larger units and compress it before uploading it to the system.

In this example, we store the Bowtie2 indices and the genome of the Zebrafish (Danio rerio) in the fish bucket. Running `ls -lh` shows that the index files are available in the current directory:

<pre>$ <b>ls -lh</b>
total 3.2G
-rw------- 1 kkayttaj csc 440M Mar 12 13:41 Danio_rerio.1.bt2
-rw------- 1 kkayttaj csc 327M Mar 12 13:41 Danio_rerio.2.bt2
-rw------- 1 kkayttaj csc 217K Mar 12 13:20 Danio_rerio.3.bt2
-rw------- 1 kkayttaj csc 327M Mar 12 13:20 Danio_rerio.4.bt2
-rw------- 1 kkayttaj csc 1.3G Mar 12 13:13 Danio_rerio.GRCz10.dna.toplevel.fa
-rw------- 1 kkayttaj csc 440M Mar 12 14:03 Danio_rerio.rev.1.bt2
-rw------- 1 kkayttaj csc 327M Mar 12 14:03 Danio_rerio.rev.2.bt2
-rw------- 1 kkayttaj csc 599K Mar 12 13:13 log
</pre>

The data is collected and compressed as a single file using the `tar` command:
```text
tar zcf zebrafish.tgz Danio_rerio*
```

The size of the resulting file is about 2 GB. The compressed file can be uploaded to the fish bucket using the command `s3cmd put`:
<pre>
$ <b>ls -lh zebrafish.tgz</b>
-rw------- 1 kkayttaj csc 9.3G Mar 12 15:23 zebrafish.tgz

$ <b>s3cmd put zebrafish.tgz s3://fish-bucket</b>
put zebrafish.tgz s3://fish-bucket
upload: 'zebrafish.tgz' -> 's3://fish-bucket/zebrafish.tgz'  [1 of 1]
 2081306836 of 2081306836   100% in   39s    50.16 MB/s  done
 
$ <b>s3cmd ls s3://fish-bucket</b>
ls s3://fish-bucket
2019-10-01 12:11 9982519261   s3://fish-bucket/zebrafish.tgz
</pre>

Uploading 2 GB of data takes some time. The uploaded file can be retrieved using the command
```text
s3cmd get s3://fish-bucket/zebrafish.tgz
```

By default, this bucket can be accessed only by project members. However, with command `s3cmd setacl`, you can make the file publicly available.

First make the fish bucket public:
```text
s3cmd setacl --acl-public s3://fish-bucket
```

Then make the zebrafish genome file public:
```text
s3cmd setacl --acl-public s3://fish-bucket/zebrafish.tgz
```

The syntax of the URL of the file:
```text
https://a3s.fi/bucket_name/object_name
```

In this case, the file would be accessible using the link
_https://a3s.fi/fish-bucket/zebrafish.tgz_
