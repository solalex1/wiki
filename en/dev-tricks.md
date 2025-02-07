# OpenIPC Wiki
[Table of Content](index.md)

Interesting tricks
------------------

### Sharing output of a command via web
```
<command> | nc seashells.io 1337
```

### Adapting syslogd to work with time zones other than GMT

Some `syslog()` implementations like musl's always send timestamps in UTC.
The following code adds a new option to `syslogd`, `-Z`, to assume incoming
timestamps are always UTC, and to adjust them to the local timezone
(of the syslogd) before logging.

```diff
Signed-off-by: Shiz <hi at shiz.me>
---
 sysklogd/syslogd.c | 23 +++++++++++++++++++----
 1 file changed, 19 insertions(+), 4 deletions(-)

diff --git a/sysklogd/syslogd.c b/sysklogd/syslogd.c
index d64ff27..159336e 100644
--- a/sysklogd/syslogd.c
+++ b/sysklogd/syslogd.c
@@ -122,6 +122,7 @@
 //usage:       "(this version of syslogd ignores /etc/syslog.conf)\n"
 //usage:	)
 //usage:     "\n	-n		Run in foreground"
+//usage:     "\n	-Z		Adjust incoming UTC times to local time"
 //usage:	IF_FEATURE_REMOTE_LOG(
 //usage:     "\n	-R HOST[:PORT]	Log to HOST:PORT (default PORT:514)"
 //usage:     "\n	-L		Log locally and via network (default is network only if -R)"
@@ -233,6 +234,8 @@ typedef struct logRule_t {
 	/*int markInterval;*/                   \
 	/* level of messages to be logged */    \
 	int logLevel;                           \
+	/* whether to adjust message timezone */\
+	int adjustTimezone;                     \
 IF_FEATURE_ROTATE_LOGFILE( \
 	/* max size of file before rotation */  \
 	unsigned logFileSize;                   \
@@ -316,6 +319,7 @@ enum {
 	OPTBIT_outfile, // -O
 	OPTBIT_loglevel, // -l
 	OPTBIT_small, // -S
+	OPTBIT_adjusttz, // -Z
 	IF_FEATURE_ROTATE_LOGFILE(OPTBIT_filesize   ,)	// -s
 	IF_FEATURE_ROTATE_LOGFILE(OPTBIT_rotatecnt  ,)	// -b
 	IF_FEATURE_REMOTE_LOG(    OPTBIT_remotelog  ,)	// -R
@@ -330,6 +334,7 @@ enum {
 	OPT_outfile     = 1 << OPTBIT_outfile ,
 	OPT_loglevel    = 1 << OPTBIT_loglevel,
 	OPT_small       = 1 << OPTBIT_small   ,
+	OPT_adjusttz    = 1 << OPTBIT_adjusttz,
 	OPT_filesize    = IF_FEATURE_ROTATE_LOGFILE((1 << OPTBIT_filesize   )) + 0,
 	OPT_rotatecnt   = IF_FEATURE_ROTATE_LOGFILE((1 << OPTBIT_rotatecnt  )) + 0,
 	OPT_remotelog   = IF_FEATURE_REMOTE_LOG(    (1 << OPTBIT_remotelog  )) + 0,
@@ -339,7 +344,7 @@ enum {
 	OPT_cfg         = IF_FEATURE_SYSLOGD_CFG(   (1 << OPTBIT_cfg        )) + 0,
 	OPT_kmsg        = IF_FEATURE_KMSG_SYSLOG(   (1 << OPTBIT_kmsg       )) + 0,
 };
-#define OPTION_STR "m:nO:l:S" \
+#define OPTION_STR "m:nO:l:SZ" \
 	IF_FEATURE_ROTATE_LOGFILE("s:" ) \
 	IF_FEATURE_ROTATE_LOGFILE("b:" ) \
 	IF_FEATURE_REMOTE_LOG(    "R:*") \
@@ -815,17 +820,23 @@ static void timestamp_and_log(int pri, char *msg, int len)
 {
 	char *timestamp;
 	time_t now;
+	struct tm nowtm = { .tm_isdst = 0 };
 
 	/* Jan 18 00:11:22 msg... */
 	/* 01234567890123456 */
 	if (len < 16 || msg[3] != ' ' || msg[6] != ' '
 	 || msg[9] != ':' || msg[12] != ':' || msg[15] != ' '
 	) {
-		time(&now);
+		now = time(NULL);
 		timestamp = ctime(&now) + 4; /* skip day of week */
 	} else {
-		now = 0;
-		timestamp = msg;
+		if (G.adjustTimezone && strptime(msg, "%b %e %T", &nowtm)) {
+			now = mktime(&nowtm) - timezone;
+			timestamp = ctime(&now) + 4; /* skip day of week */
+		} else {
+			now = 0;
+			timestamp = msg;
+		}
 		msg += 16;
 	}
 	timestamp[15] = '\0';
@@ -1130,6 +1141,10 @@ int syslogd_main(int argc UNUSED_PARAM, char **argv)
 	if (opts & OPT_loglevel) // -l
 		G.logLevel = xatou_range(opt_l, 1, 8);
 	//if (opts & OPT_small) // -S
+	if (opts & OPT_adjusttz) { // -Z
+		G.adjustTimezone = 1;
+		tzset();
+	}
 #if ENABLE_FEATURE_ROTATE_LOGFILE
 	if (opts & OPT_filesize) // -s
 		G.logFileSize = xatou_range(opt_s, 0, INT_MAX/1024) * 1024;
-- 
```
_from [sysklogd: add -Z option to adjust message timezones](http://lists.busybox.net/pipermail/busybox/2017-May/085437.html)_
