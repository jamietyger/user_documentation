# Copying data from inside of the SANBI network to outside (vice versa)

You might find yourself in the need to move data from a server inside of the SANBI network to your local machine, you vice versa.

Since `gate.sanbi.ac.za` does not contain any user data (for security purposes), you will need to create a tunnel through gate to access a server inside of the SANBI network directly. To do this, use the following command:

```bash
ssh -L<port_number>:queue.sanbi.ac.za:22 <username_of_gate>@gate.sanbi.ac.za
```
Replace <port_number> with a random 4 digit number and <username> with your SANBI system username.

Once this is done, you will be logged into the `gate` machine like a normal SSH session. You now need to a new terminal (leaving the connection to gate open in the other one) and type the following into the new terminal window (**which should be logged into your laptop NOT gate**):

```bash
scp -P <port_number> <username_of_queue>@127.0.0.1:<remote_directory_or_file_to_copy> <location_on_your_computer_to_place>
```
Replace <port_number> with the number you entered earlier, <remote_directory_or_file_to_copy> with the full directory of the data you want to copy from SANBI and <location_on_your_computer_to_place> with the directory you want to place to data on your local computer.

The reverse can be done to place a file in the SANBI network:

```bash
scp -P <port_number> <location_on_your_computer_to_place> <username_of_queue>@127.0.0.1:<remote_directory_or_file_to_copy>
````
!> If you opt to copy a folder or directory, you have to include `-r` after the `scp` command to indicate to `scp` that it must copy the contents of the folder or directory in totality. So it should look like the following: `scp -r -P ...`.

## Example

I want to copy a file `datafile.tar.gz` from my `/usr/people/edebeste` directory on the SANBI cluster. I would do the following:

1. Open a terminal and type:

```bash
ssh -L1234:queue.sanbi.ac.za:22 eugene@gate.sanbi.ac.za
```

2. Open a new terminal and type:

```bash
scp -P 1234 edebeste@127.0.0.1:/usr/people/edebeste/datafile.tar.gz ./datafile.tar.gz
```
The file will now be copied straight to my computer.

---

You can ensure the data that you have copied is not corrupt by using the `md5sum` command. This command will compute a digital signature that you can use to compare the data. To do this:

- Log onto the machine that has the data you want to copy and go to the directory that holds your data.
- Run `md5sum <data_file>` and you will get a result of some string that contains letter and numbers after some time has passed.
- Once the data is copied, go to the machine that the data was copied to and run the same `md5sum <data_file>` command on that machine.
- If the data is copied with no corruption you should see the same string that contains letter and numbers as you saw before. **It should be 100% identical**.

## Issues Solves

You for some reason you find the following (or very similar) message during the steps above:

```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:FVnauAph+p2HOgfipGQBOi1KTq4JFcivYqvW7l2Pb/g.
Please contact your system administrator.
Add correct host key in /home/pvh/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/pvh/.ssh/known_hosts:731
  remove with:
  ssh-keygen -f "/home/pvh/.ssh/known_hosts" -R "[localhost]:1234"
ECDSA host key for [localhost]:1234 has changed and you have requested strict checking.
Host key verification failed.
lost connection
```

There are two variants of this message, depending on the operating system  you are using. If you get one like the above example, you can simply run the command that you're suggested to run. It starts with `ssh-keygen -f ...`. Once that's run, you should be able to run your `scp` command again with no issues.

If you do not get a message with a suggested solution (`ssh-keygen -f ...`), then you will need to go to the following file: `/home/<your_username>/.ssh/known_hosts` and open it with a text editor. In there, you need to delete all the lines that start with `[127.0.0.1]` or `[localhost]` and you should be able to run the `scp` command again.