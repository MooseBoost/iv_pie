<div section="question">
<p>
<strong>
Writing programming interview questions hasn't made me rich. Maybe trading Apple stocks will.
</strong>
</p>
<p>
Suppose we could access yesterday's stock prices as <span words="question__stock-price__a-standard-list">an array</span>, where:
</p>
<ul>
<li>
The indices are the time in minutes past trade opening time, which was 9:30am local time.
</li>
<li>
The values are the price in dollars of Apple stock at that time.
</li>
</ul>
<p>
So if the stock cost $500 at 10:30am, <span code-inline="question__stock-price__stock-prices-yesterday-example">stockPricesYesterday[60] = 500</span>.
</p>
<p>
Write an efficient <span words="question__stock-price__function">function</span> that takes <span var="question__stock-price__stock-prices-yesterday">stockPricesYesterday</span> and returns <strong>the best profit I could have made from 1 purchase and 1 sale of 1 Apple stock yesterday.</strong>
</p>
<p>
For example:
</p>
<div code-block="question__stock-price__input-output-example" language="c" translation-highlighting="dynamic" actual-language="c">int stockPricesYesterday[6] = {10, 7, 5, 8, 11, 9};
size_t numStockPrices = 6;

getMaxProfit(stockPricesYesterday, numStockPrices);
// returns 6 (buying for $5 and selling for $11)</div>
<span words="question__stock-price__using-namespace-std"></span>
<p>
No "shorting"&#x2014;you must buy before you sell. You may not buy <em>and</em> sell in the same time step (at least 1 minute must pass).
</p>
</div>
<div section="gotchas">
<div note="" number="1" type="gotcha">
<p>
<strong>It is not sufficient to simply take the difference between the highest price and the lowest price</strong>, because the highest price may come <em>before</em> the lowest price. You must buy before you sell.
</p>
</div>
<div note="" number="2" type="gotcha">
<p>
What if the stock value <em>goes down all day</em>? In that case, the best profit will be <strong>negative</strong>.
</p>
</div>
<div note="" number="3" type="gotcha">
<p>
You can do this in <span concept="big-o-learn-more"><span complexity="n"></span> time and <span complexity="1"></span> space!</span>
</p>
</div>
</div>
<div section="breakdown">
<div note="" number="1" type="hint">
<p>
To start, try writing an example value for <span var="question__stock-price__stock-prices-yesterday">stockPricesYesterday</span> and finding the maximum profit "by hand." What's your process for figuring out the maximum profit?
</p>
</div>
<div note="" number="2" type="hint">
<p>
The <span concept="brute-force">brute force</span> approach would be to try <em>every pair of times</em> (treating the earlier time as the buy time and the later time as the sell time) and see which one is higher.
</p>
<div code-block="question__stock-price__brute-force" language="c" translation-highlighting="dynamic" actual-language="c">#define MAX(a, b) (((a) &gt; (b)) ? (a) : (b))
#define MIN(a, b) (((a) &lt; (b)) ? (a) : (b))

int getMaxProfit(const int *stockPricesYesterday, size_t length)
{
    size_t outerTime;
    int maxProfit = 0;

    // go through every time
    for (outerTime = 0; outerTime &lt; length; outerTime++) {
        size_t innerTime;

        // for every time, go through every OTHER time
        for (innerTime = 0; innerTime &lt; length; innerTime++) {

            // for each pair, find the earlier and later times
            int earlierTime = MIN(outerTime, innerTime);
            int laterTime   = MAX(outerTime, innerTime);

            // and use those to find the earlier and later prices
            int earlierPrice = stockPricesYesterday[earlierTime];
            int laterPrice   = stockPricesYesterday[laterTime];

            // see what our profit would be if we bought at the
            // min price and sold at the current price
            int potentialProfit = laterPrice - earlierPrice;

            // update maxProfit if we can do better
            maxProfit = MAX(maxProfit, potentialProfit);
        }
    }

    return maxProfit;
}</div>
<p>
But that will take <span concept="big-o-learn-more"><span complexity="n^2"></span> time,</span> since we have two nested loops&#x2014;for <em>every</em> time, we're going through <em>every other</em> time. Also, <strong>it's not correct</strong>: we won't ever report a negative profit! Can we do better?
</p>
</div>
<div note="" number="3" type="hint">
<p>
Well, we&#x2019;re doing a lot of extra work. We&#x2019;re looking at every pair <em>twice</em>. We know we have to buy before we sell, so in our <em>inner for loop</em> we could just look at every price <strong>after</strong> the price in our <em>outer for loop</em>.
</p>
<p>
That could look like this:
</p>
<div code-block="question__stock-price__smarter-brute-force" language="c" translation-highlighting="dynamic" actual-language="c">#define MAX(a, b) (((a) &gt; (b)) ? (a) : (b))
#define MIN(a, b) (((a) &lt; (b)) ? (a) : (b))

int getMaxProfit(const int *stockPricesYesterday, size_t length)
{
    size_t earlierTime;
    int maxProfit = 0;

    // go through every price and time
    for (earlierTime = 0; earlierTime &lt; length; earlierTime++) {
        size_t laterTime;

        // and go through all the LATER prices
        for (laterTime = earlierTime + 1; laterTime &lt; length; laterTime++) {
            int earlierPrice = stockPricesYesterday[earlierTime];
            int laterPrice = stockPricesYesterday[laterTime];

            // see what our profit would be if we bought at the
            // min price and sold at the current price
            int potentialProfit = laterPrice - earlierPrice;

            // update maxProfit if we can do better
            maxProfit = MAX(maxProfit, potentialProfit);
        }
    }

    return maxProfit;
}</div>
<p>
<strong>What&#x2019;s our runtime now?</strong>
</p>
</div>
<div note="" number="4" type="hint">
<p>
Well, our outer for loop goes through <em>all</em> the times and prices, but our inner for loop goes through <em>one fewer price each time</em>. So our total number of steps is the sum <span concept="summation-1-to-n"><span math="">n + (n - 1) + (n - 2) ... + 2 + 1</span></span>, which is <em>still</em> <span complexity="n^2"></span> time.
</p>
<p>
We can do better!
</p>
</div>
<div note="" number="5" type="hint">
<p>
If we're going to do better than <span complexity="n^2"></span>, we're probably going to do it in either <span complexity="nlgn"></span> or <span complexity="n"></span>. <span complexity="nlgn"></span> comes up in sorting and searching algorithms where we're recursively cutting the <span words="question__stock-price__standard-list">array</span> in half. It's not obvious that we can save time by cutting the <span words="question__stock-price__standard-list">array</span> in half here. Let's first see how well we can do by looping through the <span words="question__stock-price__standard-list">array</span> only <em>once</em>.
</p>
</div>
<div note="" number="6" type="hint">
<p>
Since we're trying to loop through the <span words="question__stock-price__standard-list">array</span> once, let's use a <span concept="greedy">greedy</span> approach, where we keep a running <span var="question__stock-price__max-profit">maxProfit</span> until we reach the end. We'll start our <span var="question__stock-price__max-profit">maxProfit</span> at $0. As we're iterating, how do we know if we've found a new <span var="question__stock-price__max-profit">maxProfit</span>?
</p>
</div>
<div note="" number="7" type="hint">
<p>
At each iteration, our <span var="question__stock-price__max-profit">maxProfit</span> is either:
</p>
<ol>
<li>the same as the <span var="question__stock-price__max-profit">maxProfit</span> at the last time step, or</li>
<li>the max profit we can get by selling at the <span var="question__stock-price__current-price">currentPrice</span>
</li>
</ol>
<p>
How do we know when we have case (2)?
</p>
</div>
<div note="" number="8" type="hint">
<p>
The max profit we can get by selling at the <span var="question__stock-price__current-price">currentPrice</span> is simply the difference between the <span var="question__stock-price__current-price">currentPrice</span> and the <span var="question__stock-price__min-price">minPrice</span> from earlier in the day. If this difference is greater than the current <span var="question__stock-price__max-profit">maxProfit</span>, we have a new <span var="question__stock-price__max-profit">maxProfit</span>.
</p>
<p>
So for every price, we&#x2019;ll need to:
</p>
<ul>
<li>keep track of the <strong>lowest price we&#x2019;ve seen so far</strong>
</li>
<li>see if we can get a <strong>better profit</strong>
</li>
</ul>
</div>
<div note="" number="9" type="hint">
<p>
Here&#x2019;s one possible solution:
</p>
<div code-block="question__stock-price__solution-before-edge-cases" language="c" translation-highlighting="dynamic" actual-language="c">#define MAX(a, b) (((a) &gt; (b)) ? (a) : (b))
#define MIN(a, b) (((a) &lt; (b)) ? (a) : (b))

int getMaxProfit(const int *stockPricesYesterday, size_t length)
{
    size_t i;
    int maxProfit = 0;
    int minPrice = stockPricesYesterday[0];

    for (i = 0; i &lt; length; i++) {
        int currentPrice = stockPricesYesterday[i];

        // ensure minPrice is the lowest price we've seen so far
        minPrice = MIN(minPrice, currentPrice);

        // see what our profit would be if we bought at the
        // min price and sold at the current price
        int potentialProfit = currentPrice - minPrice;

        // update maxProfit if we can do better
        maxProfit = MAX(maxProfit, potentialProfit);
    }

    return maxProfit;
}</div>
<p>
We&#x2019;re finding the max profit with one pass and constant space!
</p>
<p>
<strong>Are we done?</strong> Let&#x2019;s think about some edge cases. What if the stock value <em>stays the same</em>? What if the stock value <em>goes down all day</em>?
</p>
</div>
<div note="" number="10" type="hint">
<p>
If the stock price doesn't change, the max possible profit is 0. Our <span words="question__stock-price__function">function</span> will correctly return that. So we're good.
</p>
<p>
But if the value <em>goes down all day</em>, we&#x2019;re in trouble. Our <span words="question__stock-price__function">function</span> would return 0, but there&#x2019;s no way we could break even if the price always goes down.
</p>
<p>
<strong>How can we handle this?</strong>
</p>
</div>
<div note="" number="11" type="hint">
<p>
Well, what are our options? Leaving our <span words="question__stock-price__function">function</span> as it is and just returning zero is <em>not</em> a reasonable option&#x2014;we wouldn't know if our best profit was negative or <em>actually</em> zero, so we'd be losing information. Two reasonable options could be:
</p>
<ol>
<li>
<strong>return a negative profit</strong>. &#x201C;What&#x2019;s the least badly we could have done?&#x201D;</li>
<li>
<strong><span words="question__stock-price__throw-an-error">abort</span></strong>. &#x201C;We should not have purchased stocks yesterday!&#x201D;</li>
</ol>
<p>
In this case, it&#x2019;s probably best to go with option (1). The advantages of returning a negative profit are:
</p>
<ul>
<li>We <strong>more accurately answer the challenge</strong>. If profit is "revenue minus expenses", we&#x2019;re returning the <em>best</em> we could have done.</li>
<li>It's <strong>less opinionated</strong>. We'll leave decisions up to our <span words="question__stock-price__function">function</span>&#x2019;s users. It would be easy to wrap our <span words="question__stock-price__function">function</span> in a helper <span words="question__stock-price__function">function</span> to decide if it&#x2019;s worth making a purchase.</li>
<li>We allow ourselves to <strong>collect better data</strong>. It <em>matters</em> if we would have lost money, and it <em>matters</em> how much we would have lost. If we&#x2019;re trying to get rich, we&#x2019;ll probably care about those numbers.</li>
</ul>
<p>
<strong>How can we adjust our <span words="question__stock-price__function">function</span> to return a negative profit if we can only lose money?</strong> Initializing <span var="question__stock-price__max-profit">maxProfit</span> to 0 won&#x2019;t work...
</p>
</div>
<div note="" number="12" type="hint">
<p>
Well, we started our <span var="question__stock-price__min-price">minPrice</span> at the first price, so let&#x2019;s start our <span var="question__stock-price__max-profit">maxProfit</span> at the <em>first profit we could get</em>&#x2014;if we buy at the first time and sell at the second time.
</p>
<div code-block="question__stock-price__first-min-price-and-max-profit" language="c" translation-highlighting="dynamic" actual-language="c">minPrice = stockPricesYesterday[0];
maxProfit = stockPricesYesterday[1] - stockPricesYesterday[0];</div>
<p>
But we have the potential for <span words="question__stock-price__out-of-bounds-result">undefined behavior</span> here, if <span var="question__stock-price__stock-prices-yesterday">stockPricesYesterday</span> has fewer than 2 prices.
</p>
<p>
We <em>do</em> want to <span words="question__stock-price__throw-an-error">abort</span> in that case, since <em>profit</em> requires buying <em>and</em> selling, which we can't do with less than 2 prices. So, let's explicitly check for this case and handle it:
</p>
<div code-block="question__stock-price__error-requires-two-prices" language="c" translation-highlighting="dynamic" actual-language="c">assert(length &gt;= 2);  // Getting a profit requires at least 2 prices

int minPrice = stockPricesYesterday[0];
int maxProfit = stockPricesYesterday[1] - stockPricesYesterday[0];</div>
<p>
Ok, does that work?
</p>
</div>
<div note="" number="13" type="hint">
<p>
No! <strong><span var="question__stock-price__max-profit">maxProfit</span> is still always 0.</strong> What&#x2019;s happening?
</p>
</div>
<div note="" number="14" type="hint">
<p>
If the price always goes down, <span var="question__stock-price__min-price">minPrice</span> is always set to the <span var="question__stock-price__current-price">currentPrice</span>. So <span code-inline="question__stock-price__current-price-minus-min-price">currentPrice - minPrice</span> comes out to 0, which of course will always be greater than a negative profit.
</p>
<p>
When we&#x2019;re calculating the <span var="question__stock-price__max-profit">maxProfit</span>, we need to make sure we never have a case where we try <strong>both buying and selling stocks at the <span var="question__stock-price__current-price">currentPrice</span></strong>.
</p>
</div>
<div note="" number="15" type="hint">
<p>
To make sure we&#x2019;re always buying at an <em>earlier</em> price, never the <span var="question__stock-price__current-price">currentPrice</span>, let&#x2019;s switch the order around so we calculate <span var="question__stock-price__max-profit">maxProfit</span> <em>before</em> we update <span var="question__stock-price__min-price">minPrice</span>.
</p>
<p>
We'll also need to pay special attention to time 0. Make sure we don't try to buy <em>and</em> sell at time 0.
</p>
<p>
</p>
</div>
<div section="solution">
<p>
We&#x2019;ll <span concept="greedy">greedily</span> walk through the <span words="question__stock-price__standard-list">array</span> to track the max profit and lowest price so far.
</p>
<p>
For every price, we check if:
</p>
<ul>
<li>we can get a better profit by buying at <span var="question__stock-price__min-price">minPrice</span> and selling at the <span var="question__stock-price__current-price">currentPrice</span>
</li>
<li>we have a new <span var="question__stock-price__min-price">minPrice</span>
</li>
</ul>
<p>
To start, we initialize:
</p>
<ol>
<li>
<span var="question__stock-price__min-price">minPrice</span> as the first price of the day</li>
<li>
<span var="question__stock-price__max-profit">maxProfit</span> as the first profit we could get</li>
</ol>
<p>
We decided to return a <em>negative</em> profit if the price decreases all day and we can't make any money. We could have <span words="question__stock-price__thrown-an-error">aborted</span> instead, but returning the negative profit is cleaner, makes our <span words="question__stock-price__function">function</span> less opinionated, and ensures we don't lose information.
</p>
<div code-block="question__stock-price__solution" language="c" translation-highlighting="dynamic" actual-language="c">#define MAX(a, b) (((a) &gt; (b)) ? (a) : (b))
#define MIN(a, b) (((a) &lt; (b)) ? (a) : (b))

int getMaxProfit(const int *stockPricesYesterday, size_t length)
{
    int minPrice, maxProfit;
    size_t i;

    // make sure we have at least 2 prices
    assert(length &gt;= 2);

    // we'll greedily update minPrice and maxProfit, so we initialize
    // them to the first price and the first possible profit
    minPrice = stockPricesYesterday[0];
    maxProfit = stockPricesYesterday[1] - stockPricesYesterday[0];

    // start at the second (index 1) time
    // we can't sell at the first time, since we must buy first,
    // and we can't buy and sell at the same time!
    // if we started at index 0, we'd try to buy *and* sell at time 0.
    // this would give a profit of 0, which is a problem if our
    // maxProfit is supposed to be *negative*--we'd return 0.
    for (i = 1; i &lt; length; i++) {
        int currentPrice = stockPricesYesterday[i];

        // see what our profit would be if we bought at the
        // min price and sold at the current price
        int potentialProfit = currentPrice - minPrice;

        // update maxProfit if we can do better
        maxProfit = MAX(maxProfit, potentialProfit);

        // update minPrice so it's always
        // the lowest price we've seen so far
        minPrice = MIN(minPrice, currentPrice);
    }

    return maxProfit;
}</div>
</div>
<div section="complexity">
<p>
<span concept="big-o-learn-more"><span complexity="n"></span> time and <span complexity="1"></span> space.</span> We only loop through the <span words="question__stock-price__standard-list">array</span> once.
    </p>
</div>
<div section="learnings">
<p>
This one's a good example of the <span concept="greedy">greedy</span> approach in action. Greedy approaches are great because they're <em>fast</em> (usually just one pass through the input). But they don't work for every problem.
</p>
<p>
How do you know if a problem will lend itself to a greedy approach? Best bet is to try it out and see if it works. Trying out a greedy approach should be one of the first ways you try to break down a new question.
</p>
<p>
To try it on a new problem, start by asking yourself:
</p>
<p>
"Suppose we <em>could</em> come up with the answer in one pass through the input, by simply updating the 'best answer so far' as we went. What <strong><em>additional values</em></strong> would we need to keep updated as we looked at each item in our input, in order to be able to update the <strong>'best answer so far'</strong> in constant time?"
</p>
<p>
In <em>this</em> case:
</p>
<p>
The "<strong>best answer so far</strong>" is, of course, the max profit that we can get based on the prices we've seen so far.
</p>
<p>
The "<strong>additional value</strong>" is the minimum price we've seen so far. If we keep that updated, we can use it to calculate the new max profit so far in constant time. The max profit is the larger of:
</p>
<ol>
<li>
The previous max profit
</li>
<li>
The max profit we can get by selling now (the current price minus the minimum price seen so far)
</li>
</ol>
<p>
Try applying this greedy methodology to future questions.
</p>
</div>
</div>