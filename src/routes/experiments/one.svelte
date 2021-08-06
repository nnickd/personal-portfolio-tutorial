<!-- <script>
    import One from '../../components/experiments/One.svelte'
</script>

<One></One> -->

<script>
export const prerender = true;

import { onMount } from 'svelte';

let p5;

onMount(async () => {
    const module = await import('p5-svelte');
    p5 = module.default;
});

// let width = 55;
// let height = 55;

// const sketch = (p5) => {
//   p5.setup = () => {
//     p5.createCanvas(400, 400);
//   };

//   p5.draw = () => {
//     p5.ellipse(p5.width / 2, p5.height / 2, width, height);
//   };
// };

// function Elemental(type, point, attrs) {}
function Point(x, y, radius, mass, limit, _color, lineDist, forceDist, gravity) {
    this.pos = p5.createVector(x, y);
    this.vel = p5.createVector(0, 0);
    this.acc = p5.createVector(0, 0);

    this.limit = limit || 6;
    this.radius = radius || 30;
    this.mass = mass || 10;
    this.color = _color || 69;
    this.lineDist = lineDist || 100;
    this.forceDist = forceDist || 100;
    this.gravity = gravity || 0.01;
    }

    Point.prototype.tick = function (toroidial = false) {
        this.vel.add(this.acc);
        this.vel.limit(this.limit);
        this.pos.add(this.vel);
        this.acc.mult(0);

        if (toroidial) {
            if (this.pos.x < -p5.width / 2 + this.radius) {
                this.pos.x = p5.width / 2 - this.radius;
            }
            if (this.pos.x > p5.width / 2 - this.radius) {
                this.pos.x = -p5.width / 2 + this.radius;
            }
            if (this.pos.y < -p5.height / 2 + this.radius) {
                this.pos.y = p5.height / 2 - this.radius;
            }
            if (this.pos.y > p5.height / 2 - this.radius) {
                this.pos.y = -p5.height / 2 + this.radius;
            }
        } else {
            if (this.pos.x < -p5.width / 2 + this.radius) {
                this.vel.x *= -1;
                this.pos.x = -p5.width / 2 + this.radius;
            }
            if (this.pos.x > p5.width / 2 - this.radius) {
                this.vel.x *= -1;
                this.pos.x = p5.width / 2 - this.radius;
            }
            if (this.pos.y < -p5.height / 2 + this.radius) {
                this.vel.y *= -1;
                this.pos.y = -p5.height / 2 + this.radius;
            }
            if (this.pos.y > p5.height / 2 - this.radius) {
                this.vel.y *= -1;
                this.pos.y = p5.height / 2 - this.radius;
            }
        }
    };

    Point.prototype.draw = function () {
        p5.push();
        p5.translate(p5.width / 2, p5.height / 2);
        p5.fill(this.color);
        p5.ellipse(this.pos.x, this.pos.y, this.radius, this.radius);
        p5.pop();
    };

    Point.prototype.force = function (force) {
        this.acc.add(force.copy().div(this.mass));
    };

    function Gravity(p, p2) {
        let target = p2.pos.copy().sub(p.pos);
        let dist = target.mag();

        if (dist <= (p.forceDist + p2.forceDist) / 2 && dist > p.radius + p2.radius) {
            target.normalize();
            let gravity = (p.gravity + p2.gravity) / 2;
            let force = (gravity * p.mass * p2.mass) / (dist * dist);
            target.mult(force);
            p.force(target);
            target.mult(-1);
            p2.force(target);
        }
    }

    function PointLine(p, p2) {
        let dist = p5.Vector.dist(p.pos, p2.pos);

        if (dist <= (p.lineDist + p2.lineDist) / 2) {
            p5.push();
            p5.translate(p5.width / 2, p5.height / 2);
            p5.stroke(98);
            p5.line(p.pos.x + p.vel.x, p.pos.y + p.vel.y, p2.pos.x + p2.vel.x, p2.pos.y + p2.vel.y);
            p5.pop();
        }
    }

    function* polygon(x, y, radius, npoints) {
        let angle = p5.TWO_PI / npoints;
        let n = 0;
        for (let a = 0; a < p5.TWO_PI; a += angle) {
            let sx = x + p5.cos(a) * radius;
            let sy = y + p5.sin(a) * radius;
            if (npoints >= ++n) {
                let vec = p5.createVector(sx, sy);
                // vec.setHeading(0);
                yield vec;
            }
        }
    }

    function polyception(x, y, nlevels, ops) {
        let points = [];
        let pmap = [];

        for (let i = 0; i < nlevels; i++) {
            pmap.push([]);
        }

        for (let i = 0; i < nlevels; i++) {
            let radius = ops.radius[i] || ops.radius;
            let npoints = ops.npoints[i] || ops.npoints;
            let pradius = ops.pradius[i] || ops.pradius;
            let mass = ops.mass[i] || ops.mass;
            let limit = ops.limit[i] || ops.limit;
            let pcolor = ops.color[i] || ops.color;
            let lineDist = ops.lineDist[i] || ops.lineDist;
            let forceDist = ops.forceDist[i] || ops.forceDist;
            let gravity = ops.gravity[i] || ops.gravity;

            let _p = pmap[i - 1];
            let npmax = i > 0 ? _p.length : 1;
            for (let np = 0; np < npmax; np++) {
                let _x = i > 0 ? _p[np].x : x;
                let _y = i > 0 ? _p[np].y : y;
                for (let poly of polygon(_x, _y, radius, npoints)) {
                    let p = new Point(
                        poly.x,
                        poly.y,
                        pradius,
                        mass,
                        limit,
                        pcolor,
                        lineDist,
                        forceDist,
                        gravity
                    );
                    pmap[i].push(p.pos.copy());
                    points.push(p);
                }
            }
        }

        console.log(points, points.length);
        return points;
    }

    function PolyOps(npoints, radius, pradius, mass, limit, _color, lineDist, forceDist, gravity) {
        this.radius = radius;
        this.npoints = npoints;
        this.pradius = pradius;
        this.mass = mass;
        this.limit = limit;
        this.color = _color;
        this.lineDist = lineDist;
        this.forceDist = forceDist;
        this.gravity = gravity;
    }

    let points = [];
    let groups = [];
    let loopy = true;
    let toroidial = false;
    let upToggle = true;


const sketch = (p5) => {

    

    p5.setup = () => {
        console.log(this, p5)
        let canv = p5.createCanvas(p5.windowWidth, p5.windowHeight);
        // canv.mousePressed(mousePressed);
        // canv.keyPressed(keyPressed)
        // p5.background(180);
        let six = new PolyOps(
        ([6, 6]),
        ([160, 80]),
        (10),
        ([1000, 10]),
        (100),
        (66),
        (90),
        (90),
        (0.01)
        );
            console.log(six);
        
    let _points = polyception( 0, 0, 2, six);
    points = points.concat(_points);

    points.push(new Point(0, 100, 30, 100, 5, 66, 100, 100, .1))
    points.push(new Point(0, -100, 30, 100, 5, 66, 100, 100, .1))
    points.push(new Point(-100, -100, 30, 100, 5, 66, 100, 100, .1))
    points.push(new Point(100, 100, 30, 100, 5, 66, 100, 100, .1))

    };

    p5.draw = () => {
        console.log(p5)
        p5.background(180);

        // for (let points of groups) {

        for (let i = 0; i < points.length; i++) {
            for (let j = i; j < points.length; j++) {
                if (i != j) {
                    let p = points[i];
                    let p2 = points[j];
                    Gravity(p, p2);
                    PointLine(p, p2);
                }
            }
        }
        // }
        // for (let points of groups) {
        for (let point of points) {
            point.tick(toroidial);
            point.draw();
        }
        // }
    };

    function keyPressed ()  {
        if (p5.keyCode == p5.ENTER) {
            loopy = !loopy;
            loopy ? p5.loop() : p5.noLoop();
        }
        if (p5.keyCode == p5.DELETE) {
            points = [];
            groups = [];
        }
        if (p5.keyCode == p5.SHIFT) {
            toroidial = !toroidial;
        }
        if (p5.keyCode == p5.UP_ARROW) {
            upToggle = !upToggle;
        }

    }

    function mousePressed ()  {
        debugger
    let mx = p5.mouseX - p5.width / 2;
    let my = p5.mouseY - p5.height / 2;

    // let six = new PolyOps(
    //     // @ts-ignore
    //     (npoints = [6, 6]),
    //     // @ts-ignore
    //     (radius = [160, 80]),
    //     // @ts-ignore
    //     (pradius = 10),
    //     // @ts-ignore
    //     (mass = [1000, 10]),
    //     // @ts-ignore
    //     (limit = 100),
    //     // @ts-ignore
    //     (_color = 66),
    //     // @ts-ignore
    //     (lineDist = 90),
    //     // @ts-ignore
    //     (forceDist = 90),
    //     // @ts-ignore
    //     (gravity = 0.01)
    // );
    let six = new PolyOps(
        ([6, 6]),
        ([160, 80]),
        (10),
        ([1000, 10]),
        (100),
        (66),
        (90),
        (90),
        (0.01)
        );
            console.log(six);
        
    let _points = polyception(upToggle ? mx : 0, upToggle ? my : 0, 2, six);
    points = points.concat(_points);

    if (!loopy) {
        for (let point of points) {
            point.draw();
        }
    }
    // groups.push(_points);

    }
};


// function mouseDragged() {
//   mousePressed();
// }

</script>


<svelte:component this={p5} {sketch} />