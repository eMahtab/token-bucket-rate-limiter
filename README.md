# Token Bucket Rate Limiter
The Token Bucket algorithm is considered more flexible and efficient compared to the Fixed Window and Sliding Window approaches.

```java
import java.time.Instant;
import java.util.concurrent.atomic.AtomicInteger;

public class TokenBucketRateLimiter {
    private final int capacity;            // Maximum tokens the bucket can hold
    private final int refillRate;          // Number of tokens added per second
    private AtomicInteger tokens;          // Current number of tokens
    private long lastRefillTimestamp;      // Time when the bucket was last refilled

    public TokenBucketRateLimiter(int capacity, int refillRate) {
        this.capacity = capacity;
        this.refillRate = refillRate;
        this.tokens = new AtomicInteger(capacity); // Start with a full bucket
        this.lastRefillTimestamp = System.currentTimeMillis();
    }

    // Method to refill tokens based on elapsed time
    private void refill() {
        long now = System.currentTimeMillis();
        long timeElapsed = now - lastRefillTimestamp;
        // Calculate the number of tokens to add based on elapsed time
        int tokensToAdd = (int) (timeElapsed * refillRate / 1000);
        if (tokensToAdd > 0) {
            int newTokens = Math.min(tokens.get() + tokensToAdd, capacity);
            tokens.set(newTokens);
            lastRefillTimestamp = now;
        }
    }

    // Method to check if a request can be allowed
    public synchronized boolean isAllowed() {
        refill();
        if (tokens.get() > 0) {
            tokens.decrementAndGet(); // Consume a token
            return true; // Request is allowed
        }
        return false; // Request is rejected
    }

    public static void main(String[] args) throws InterruptedException {
        // Create a TokenBucket with a capacity of 10 tokens and a refill rate of 1 token per second
    	TokenBucketRateLimiter bucket = new TokenBucketRateLimiter(10, 1);
        // Simulate 27 requests
        for (int i = 1; i <= 27; i++) {
        	Instant now = Instant.now();
            if (bucket.isAllowed()) {
                System.out.println("Request " + i + ": Allowed at "+ now);
            } else {
                System.out.println("Request " + i + ": Rejected at "+ now);
            }
            Thread.sleep(200); // Wait 200 milliseconds between requests
        }
    }
}
```

**Code Execution Output :**

```
Request 1: Allowed at 2024-11-09T05:51:18.887464500Z
Request 2: Allowed at 2024-11-09T05:51:19.108508900Z
Request 3: Allowed at 2024-11-09T05:51:19.309283300Z
Request 4: Allowed at 2024-11-09T05:51:19.524123600Z
Request 5: Allowed at 2024-11-09T05:51:19.724932600Z
Request 6: Allowed at 2024-11-09T05:51:19.926665600Z
Request 7: Allowed at 2024-11-09T05:51:20.129833200Z
Request 8: Allowed at 2024-11-09T05:51:20.331960200Z
Request 9: Allowed at 2024-11-09T05:51:20.534098400Z
Request 10: Allowed at 2024-11-09T05:51:20.748518600Z
Request 11: Allowed at 2024-11-09T05:51:20.949546400Z
Request 12: Allowed at 2024-11-09T05:51:21.151410Z
Request 13: Rejected at 2024-11-09T05:51:21.353294800Z
Request 14: Rejected at 2024-11-09T05:51:21.554324500Z
Request 15: Rejected at 2024-11-09T05:51:21.757375100Z
Request 16: Allowed at 2024-11-09T05:51:21.961856500Z
Request 17: Rejected at 2024-11-09T05:51:22.164430200Z
Request 18: Rejected at 2024-11-09T05:51:22.365078200Z
Request 19: Rejected at 2024-11-09T05:51:22.568775Z
Request 20: Rejected at 2024-11-09T05:51:22.772719700Z
Request 21: Allowed at 2024-11-09T05:51:22.986887100Z
Request 22: Rejected at 2024-11-09T05:51:23.191225200Z
Request 23: Rejected at 2024-11-09T05:51:23.395519600Z
Request 24: Rejected at 2024-11-09T05:51:23.598851900Z
Request 25: Rejected at 2024-11-09T05:51:23.800981100Z
Request 26: Allowed at 2024-11-09T05:51:24.001809Z
Request 27: Rejected at 2024-11-09T05:51:24.205778600Z
```
