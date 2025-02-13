# John The Ripper

## Cracking Modes

```bash
$john --format=sha256/afs/bfegg/ hashes_to_crack.txt
```

John will output the cracked passwords to the console and the file "john.pot" (`~/.john/john.pot`) to the current user's home directory. Furthermore, it will continue cracking the remaining hashes in the background, and we can check the progress by running the `john --show` command.
