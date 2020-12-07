# SYN-Attack-Suppressor-Routine Dashboard
A web dashboard for monitoring the SYN-Attack-Suppressor-Routine
  
```
#!/bin/bash
#
# SYN_RECV_Dashboard.sh
#
# Build the dashboard for the SYN-Attack-Suppressor-Routine
#
#--------------------------------------------------------------#
#       --- SYN-Attack-Suppressor-Routine Dashboard ---        #
#                                                              #
# The following script creates a dashboard to view a server's  #
# NETSTAT along with any IPs that are currently being blocked  #
# by the SYN-Attack-Suppressor-Routine.                        #
#                                                              #
# Set your web page path in the 'webpgpath=' parameter below.  #
# Access the dashboard with https://yourDN.TDL/suppressor.html #
#                                                              #
# Run this cronjob every minute to create web page:            #
# */1 * * * * /path/to/SYN_RECV_Dashboard.sh                   #
#--------------------------------------------------------------#

#top

webpgpath="/var/www/html"      # This needs to point to your server's html folder

filePath=${0%/*}  # current file path
webpgnm="/suppressor.html"

stathighlight="#ff1493"
fontsz="2"
hfontsz="2"
bgclr=#EBEADF
fontcolor=#00000040
titlecolor=#f0f0f0
netstatswidth=230
titlecolor=#f0f0f0


cat /dev/null > $webpgpath/$webpgnm  # clear html file

echo "<html><head><title>SASR Dashboard</title></head><body bgcolor=\"$bgclr\" TEXT=\"$fontcolor\" LANG=\"en-US\" DIR=\"LTR\">" > $webpgpath/$webpgnm

##'-------------------'
## BEGIN Top date line
##'-------------------'
echo "<font face=\"Verdana\" size=$fontsz color=\"$fontcolor\"><b>&nbsp;" >> $webpgpath/$webpgnm
SRDvar=$(date | cut -c 1-10 && date | cut -c 25-28 && echo "-" && date | cut -c 12-23)
SRDvar=${SRDvar//:01 / } && SRDvar=${SRDvar//:02 / } && SRDvar=${SRDvar//:03 / }  # remove seconds position
#echo "$SRDvar" - ${0##*/} >> $webpgpath/$webpgnm
echo "$SRDvar" >> $webpgpath/$webpgnm
echo "</font></b><br>" >> $webpgpath/$webpgnm
##'-------------------'
##  END Top date line
##'-------------------'

echo "<table><tr valign=\"top\"><td>" >> $webpgpath/$webpgnm  # overlaying table begin

##'--------------------'
## BEGIN Netstats table
##'--------------------'
echo "<table border=\"1\" cellpadding=\"2\" cellspacing=\"0\"><tr><td bgcolor=\"$titlecolor\">
<font face=\"Verdana\" size=$fontsz><b>Netstats</b></font></td></tr>
<tr><td><font face=\"Courier New\" size=$fontsz color=\"$fontcolor\"><b>" >> $webpgpath/$webpgnm
netstat -vatnp > $filePath/SRDtempl.txt

sed -i "/tcp6/d" $filePath/SRDtempl.txt            # remove tcp6 lines
sed -i "/python2.7/d" $filePath/SRDtempl.txt       # remove bitmessage python2.7 messages
sed -i "/nginx\: worker/d" $filePath/SRDtempl.txt  # remove nginx: worker messages
sed -i "/TIME_WAIT/d" $filePath/SRDtempl.txt       # remove TIME_WAIT messages
sed -i 's/ /\&nbsp;/g' $filePath/SRDtempl.txt
sed -i 's/ //g' $filePath/SRDtempl.txt
sed -e 's/$/<br>/' $filePath/SRDtempl.txt >> $webpgpath/$webpgnm
echo "</font></td></tr></table><br>" >> $webpgpath/$webpgnm
##'--------------------'
##  END Netstats table
##'--------------------'


echo "</td><td>" >> $webpgpath/$webpgnm  # MIDDLE: vertical divider between Netstats and MY_SYN_DROP tables


##'-----------------------'
## BEGIN MY_SYN_DROP table
##'-----------------------'
echo "<table border=\"1\" cellpadding=\"2\" cellspacing=\"0\"><tr><td bgcolor=\"$titlecolor\">
<font face=\"Verdana\" size=$fontsz><b>MY_SYN_DROP</b></font></td></tr>
<tr><td><font face=\"Courier New\" size=$fontsz color=\"$fontcolor\"><b>" >> $webpgpath/$webpgnm

echo "$(iptables -L MY_SYN_DROP -n)" > $filePath/SRDtempl.txt
sed -i 's/$/<br>/' $filePath/SRDtempl.txt   # add <br> to end of every rec
sed -i '1,2d' $filePath/SRDtempl.txt        # delete lines 1 & 2
awk '{ print $4 }' $filePath/SRDtempl.txt > $filePath/SRDtemp2.txt
sed -i 's/$/<br>/g' $filePath/SRDtemp2.txt
cat $filePath/SRDtemp2.txt >> $webpgpath/$webpgnm

echo "</font></td></tr></table>" >> $webpgpath/$webpgnm
##'-----------------------'
##  END MY_SYN_DROP table
##'-----------------------'


echo "</td></tr></table>" >> $webpgpath/$webpgnm  # overlaying table end


echo "</body></html>" >> $webpgpath/$webpgnm

rm $filePath/SRDtempl.txt
rm $filePath/SRDtemp2.txt

exit 0
```
