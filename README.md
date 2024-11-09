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
