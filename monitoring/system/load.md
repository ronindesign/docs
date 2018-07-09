## Load Averages

1, 5, 15 min load averages are not an average over those intervals.\
They are predicted values given some historical set of data.

**Example:**

With CPU @ 100%, after 1 minute we would expect 1-min load avg. of 1.0.\
However, instead we have load around 0.62 (the predicted value after 1 min of data).

![Load Averages 1, 5, 15 Min](http://www.brendangregg.com/blog/images/2017/loadavg.png)

Reference:

http://www.brendangregg.com/blog/2017-08-08/linux-load-averages.html
http://www.teamquest.com/import/pdfs/whitepaper/ldavg1.pdf
