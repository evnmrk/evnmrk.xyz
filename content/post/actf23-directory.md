---
title: "ångstromCTF 2023 - Directory Writeup"
date: 2023-09-15T22:28:55-07:00
tags: ["ångstromCTF", "CTF", "perl", "web"]
categories: ["CTF Writeup"]
draft: false
---

# Introduction

This (very late) writeup is for the [2023 ångstromCTF](https://2023.angstromctf.com/) "directory" web challenge. This challenge involved a webpage with 5000 directories, with one directory holding the flag. I wrote a simple perl script to enter each directory until the flag is found. Note that every directory without the flag instead holds the text, "your flag is in another file".

# Perl Script
```perl
#!/usr/bin/perl

use strict;
use warnings;
use LWP::Simple;

my $URL = "https://directory.web.actf.co/";

foreach (0 .. 5000){
    my $new_URL = $URL . $_ . '.html';
    my $response = get $new_URL;
    print "$_\n";
    unless ( $response eq "your flag is in another file") {
        print $response;
        last;
    }   
}
```

# Result

The script found the flag in about 5 minutes. The script is not very fast, but it gets the job done.
