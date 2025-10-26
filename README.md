# Gaming IP Blocklists

For blocking gaming ip addresses, the two imporant (i.e. usable) files are:
- ***blocklist_ip4_cidr***
- ***blocklist_ip6_cidr***

*blocklist_ip4_cidr* is the more curated list, as duplicates and subnets have been removed.  However, the removal process is somewhat crude as it is text based via a shell script, and thus subnet identification is not thorough.  *blocklist_ip6_cidr* is raw and thus unsorted and likely contains some duplicates and subnets.

The remaining files are informational:
1.  ***blocklist_corps*** - list of the gaming corporations which are mined for ip address blocks.
2.  ***blocklist_ans_id*** - list of the ASN id's listed for all of the corporations.
3.  ***blocklist_map*** - tree-like representation of the corporations, their registered ASN id', and their registered ip blocks.
4.  ***blocklist_ip4_stats*** - CSV table of all ipv4 addresses which allows calculating some statistics.  The file *blocklist_ip4_cidr* is derived by extracting and curating the IP column of this table.

## List Creation Algorithm
This approach/algorithm was inspired by a post I read somewhere which was looking up the corporate ip blocks of a gaming corporations for whitelisting.  I have somewhat inverted the idea into blocklists:  I created a list of top gaming corporations and lookup their associated ANS id's and the ip blocks associated with them.

I am intentionally not giving a detailed explanation of the data acquisition algorithm as I implement several degrees of *trickiness* to acquire the data while avoiding being blocked/banned.

The curation of the ip4 list is as follows:
1. A row number column is prepended to the ip4_stats csv file (to allow later reordering into the original order).
2. The file is sorted on the IP column (which contains ipv4 address in CIDR format), which conveniently orders identical ipv4 addresses with descending CIDR block sizes (i.e. ascending bitmasks).
3. The the IP column is extracted and the CIDR mask is truncated, yielding a list of [sorted] ipv4 addresses.
4. 'uniq -d' is run to identify duplicate ipv4 addresses.
5. For each duplicate ipv4 address, grep is run on the sorted table to identify lines containing the ipv4 address.  The first line of occurance is ignored (it has either same or larger block size than subsequent lines), but all subsequent occurance lines are removed.
6. The IP column is then extracted from the sorted table, the row header is removed, which yields the *blocklist_ip4_cidr* file.
7. After each of the duplicate/subnet lines have been removed, the table is resorted based on the original row numbers to achive the original order minus the duplicate/subnet lines.  The row number column is then removed, yielding the final *blocklist_ip4_stats* file.

## Summary Statistics
Here are summary statistics of the initial ip4 list derived from *blocklist_ip4_stats*.  This is only for conceptual information.  I do not intend to update this regularly, but I will likely update it if the corporation list changes.
```
                          CORP  ADDRESS_COUNT
                       Tencent  14,104,064
                       NetEase     391,936
        Blizzard Entertainment      99,584
Sony Interactive Entertainment      60,672
                        Roblox      47,360
            Twitch Interactive      35,328
                       Ubisoft      28,672
                        Unreal      25,344
          Take-Two Interactive      23,040
                         Valve      17,920
                      Nintendo       4,352
                    Epic Games       3,584
                    CD PROJEKT         256
                                          
                         TOTAL  14,842,112
```
The ip4 list blocks about 700k IP's if one excludes Tencent.  I've yet to figure out a way to identify the subset of Tencent's IP's which are dedicated to gaming, but blocking all Tencent IP's works for me.  Conversely, Sony conducting their gaming business under the "*Sony Interactive Entertainment"* name greatly simplifies identifying their gaming IP's.

## Disclaimer
I make zero promises about the accuracy of these lists.  They work for me, and that is my major accuracy criteria.
