#define _POSIX_C_SOURCE 200809L
#include &lt;stdio.h&gt;
#include &lt;fcntl.h&gt;
#include &lt;unistd.h&gt;

/* Copy src to dst with the read() and write() system calls, using a buffer of
bufsize bytes. Returns the number of read() calls made, or -1 on error. */
long copy_file(const char *src, const char *dst, size_t bufsize)
{
char buf[8192];
int in = open(src, O_RDONLY);
if (in &lt; 0) return -1;
int out = open(dst, O_WRONLY | O_CREAT | O_TRUNC, 0644);
if (out &lt; 0) { close(in); return -1; }
long calls = 0;
ssize_t n;
// TODO 1 (completed): loop read/write until EOF or error
while (1) {
n = read(in, buf, bufsize);
calls++; /* count every read call */
if (n &lt;= 0) break; /* 0 = end of file, -1 = error */
ssize_t done = 0;
while (done &lt; n) { /* write exactly the n bytes read */
ssize_t w = write(out, buf + done, n - done);
if (w &lt; 0) { close(in); close(out); return -1; }
done += w;
}
}

CSA0417 – Operating Systems – Assignment 1 | Page 19

close(in);
close(out);
return calls;
}
int main(void)
{
FILE *f = fopen(&quot;records.dat&quot;, &quot;w&quot;); /* a 10,000-byte test file */
for (int i = 0; i &lt; 10000; i++) fputc(&#39;A&#39; + i % 26, f);
fclose(f);

size_t sizes[] = {1, 512, 4096};
for (int i = 0; i &lt; 3; i++)
printf(&quot;buffer %4zu bytes: %5ld read() calls\n&quot;, sizes[i],
copy_file(&quot;records.dat&quot;, &quot;copy.dat&quot;, sizes[i]));
unlink(&quot;records.dat&quot;);
unlink(&quot;copy.dat&quot;);
return 0;
}
