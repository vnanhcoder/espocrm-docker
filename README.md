# Espocrm docker

## add host file

```
127.0.0.1 espo.crm
127.0.0.1 mongo-express.crm
127.0.0.1 phpmyadmin.crm
127.0.0.1 redisinsight.crm
```
let s = 10*3600,
    t = 56 * 3600,
    d = 0;
let timelines = [
    [
        [8.5*3600, 12 * 3600],
        [13.5*3600, 18 * 3600]
    ],
    [
        [8.5*3600, 12 * 3600],
        [13.5*3600, 19 * 3600]
    ],
    [
        [8.5*3600, 12 * 3600],
        [13.5*3600, 19 * 3600],
    ],
    [
        [8.5*3600, 12 * 3600],
        [13.5*3600, 19 * 3600],
    ],
    [
        [8.5*3600, 12 * 3600],
        [13.5*3600, 19 * 3600],
    ],
    [
        [8.5*3600, 14 * 3600]
    ],
    [[0, 0]]
]
i = 0;
do {
    for (let i = 0; i < timelines.length; i++) {
        for (let index = 0; index < timelines[i].length; index++) {
            let [start, finish] = timelines[i][index];
            
            if (s < start) {
                s = start;
            }
            
            if (s + t < finish) {
                s += t;
                t = 0;
                break;
            }
    
            if (s > finish) {
                continue;
            }
    
            t = t - finish + s
            s = finish;
        }

        if (t >= 0) {
            d++;
            s = t > 0 ? 0 : s;
            if(t <=0) {
                break;
            }
        }
    }
    
}while(t > 0);
console.log(s/3600, d, t)