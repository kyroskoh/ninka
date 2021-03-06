Contact information
===================

Any feedback will be appreciated. You can email us at Daniel M. German
<dmg@uvic.ca> and Yuki Manabe <y-manabe@ist.osaka-u.ac.jp>

Introduction
------------

Ninka is license identification tool that identifies the license(s)
under which a given source file is made available.

This tool uses a source file as input and outputs the licenses
identified within that file.

If you need to know the detail of Ninka, please see the following paper:

Daniel M. German, Yuki Manabe and Katsuro Inoue. A sentence-matching
method for automatic license identification of source code files. In
25nd IEEE/ACM International Conference on Automated Software
Engineering (ASE 2010). You can email me (dmg@uvic.ca) for a copy or
download it from http://turingmachine.org/~dmg/papers/dmg2010ninka.pdf.

If you use Ninka for research purposes, we would appreciate you cite
the above paper.

Contributors
------------

- Paul Clough for his code to split sentences
- Anthony Kohan for writing the excel and sqlite backends
- Armijn Hemel from Tjaldur Software Governance Solutions for multiple bug reports and suggestions
- René Scheibe for modularizing the code

License
-------

  Ninka is licensed under the GPLv2+:

    Copyright (C) 2009-2014  Yuki Manabe and Daniel M. German

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as
    published by the Free Software Foundation; either version 2 of the
    License, or (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.

  Ninka::SentenceExtraxtor is a derivative work of the rule-based sentence
  splitter script by Paul Paul Clough.

  comments is based on a program to remove comments by Jon Newman.

Requirements
------------

- Perl version 5 or above
- for ninka-excel: Perl module Spreadsheet::WriteExcel
  https://metacpan.org/release/Spreadsheet-WriteExcel
- for ninka-sqlite: Perl module DBD::SQLite
  https://metacpan.org/release/DBD-SQLite

* How to install

  1. Unpack the distribution in a directory.
  2. Optional: Build and install comments (make sure it is somwehere in the path) (see directory comments)

Usage
-----

    ninka [options] filename

Available options:

    -i create intermediary files
    -v verbose

Example:

    ninka -i foo.c

It will create five files:

  1. foo.c.comments: extracted the first comments blocks, where
     the license is usually included
  2. foo.c.sentences: creates the list of sentences in the license
     statement
  3. foo.c.goodsent: contains sentences that are likely to be part of
     a license statement
  4. foo.c.badsent: contains the sentences that are not part of
     foo.c.goodsent
  5. foo.c.senttok: Each sentence in *.goodsent is converted into a
     tokenized sentence (or unmatched, when none matches)
  6. foo.c.license: List of licenses found in the file. Its contains a
     single line with 3 fields (semicolon delimited):
     - Licenses
     - Unmatched sentences in *.senttok that were not matched

The files are not required for Ninka's functionality. But they can help
to debug license detection issues.

Ninka model
-----------

Ninka uses a pipe-model. Each stage of the pipe does something very specific:

1. Comment extractor

    - Module: Ninka::CommentExtractor

    - Purpose: Extracts top comments of source code.
               If no comment extractor is known for the language,
               then extracts top lines from source (currently 700)

    - Output: <filename>.comments

2. Split sentences in comments

     - Module: Ninka::SentenceExtractor

     - Purpose: Ninka works by matching sentences of licenses,
                hence it needs to properly break text into sentences.

     - Output: <filename>.sentences

3. Filter "good" sentences

     - Module: Ninka::SentenceFilter

     - Purpose: Some sentences are related to a license, some are not.
                It is valuable to know if a file contains lines that look like
                a license or not (e.g. to know that a file has no license).

     - Output: <filename>.goodsent and <filename>.badsent

4. Tokenize sentences

     - Module: Ninka::SentenceTokenizer

     - Purpose: It creates a file that corresponds to the recognized sentence tokens.
                For each sentence, it outputs its sentence token, or unknown otherwise.

     - Output: <filename>.senttok

5. Match sentences to licenses

     - Module: Ninka::LicenseMatcher

     - Purpose: It looks at the sentence tokens and outputs the licenses found.

     - Output: <filename>.license

The script ninka takes care of all these steps, and optionally creates
intermediary files, and writes to the stdout the licenses found.

------

How to read the output:

Assume, for example, this output:

    eq.c;MITX11noNotice;1;2;2;6;0;Copyright,-1,-1,DualLicenseIntention,GPLorOpenBSDTypeVer2,BSDpre,BSDcondSource,BSDcondBinary

So Ninka detects all the sentences, including the MIT variant, it
finds the GPL bsd intention. But the license is not really BSD.

The disclaimers are not what you expect. Now, in all fairness, maybe
this is another license.


Let me translate the output for you:

file: eq.c;
License(s) found: MITX11noNotice


;1;2;2;6;0;
Found 1 license
Composed of 2 lines (tokens)
2 tokens were ignored
6 tokens were not mached: Copyright,-1,-1,DualLicenseIntention,GPLorOpenBSDTypeVer2,BSDpre,BSDcondSource,BSDcondBinary (-1 indicates where a match happened)
0 tokens were unknown


Another example:

nsAccessibilityUtils.cpp;MPLv1_1;1;1;3;7;2;UNKNOWN,MPL1_1_GPL2_LGPL2_1intentionVer0,1,-1,-1,MPLsee,Copyright,-1,Altern,UNKNOWN,MPLoptionNOTGPLVer0,MPLoptionIfNotDelete3licsVer0,licenseBlockEnd

License matched:MPLv1_1;
One license: 1;
Composed of one token: 1;
3 token were ignored 3;
7 tokens were matched but not recognized as a license: UNKNOWN,MPL1_1_GPL2_LGPL2_1intentionVer0,1,-1,-1,MPLsee,Copyright,-1,Altern,UNKNOWN,MPLoptionNOTGPLVer0,MPLoptionIfNotDelete3licsVer0,licenseBlockEnd
2 of those tokens were unknown
