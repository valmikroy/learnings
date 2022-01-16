# Poisson Process

The Poisson process is one of the most widely-used counting processes. It is usually used in scenarios where we are counting the occurrences of certain events that appear to happen at a certain rate, but completely at random (without a certain structure).

Above example of compounding is exponential growth function while Poission process has an exponential decay function like following which reaches to 1 along with increase in value of `t`.

This is similar to above function converts into

$$
-\frac1{rate} log(rand(0..1))
$$

```
#!/usr/bin/env ruby

rate = 1000000 e = 2.7182
sum = 0

rate.times do |a|

    interval = Math.log(rand(),e)/rate * -1

    # over here we can add a multiplication factor
    sum += interval

end

#this sum is always close to 1

puts sum
```

**Reference**&#x20;

[Basic Concepts of the Poisson Process](https://www.probabilitycourse.com/chapter11/11\_1\_2\_basic\_concepts\_of\_the\_poisson\_process.php)
