I am thinking of Conway's Game of Life recently.  The latest Adabox016 had a
16x32 display and one of the demos was the game of life.  They had a pretty nice
implementation in python, with the central part being:

    # Conway's "Game of Life" is played on a grid with simple rules, based
    # on the number of filled neighbors each cell has and whether the cell itself
    # is filled.
    #   * If the cell is filled, and 2 or 3 neighbors are filled, the cell stays
    #     filled
    #   * If the cell is empty, and exactly 3 neighbors are filled, a new cell
    #     becomes filled
    #   * Otherwise, the cell becomes or remains empty
    #
    # The complicated way that the "m1" (minus 1) and "p1" (plus one) offsets are
    # calculated is due to the way the grid "wraps around", with the left and right
    # sides being connected, as well as the top and bottom sides being connected.
    #
    # This function has been somewhat optimized, so that when it indexes the bitmap
    # a single number [x + width * y] is used instead of indexing with [x, y].
    # This makes the animation run faster with some loss of clarity. More
    # optimizations are probably possible.

     def apply_life_rule(old, new):
         width = old.width
         height = old.height
         for y in range(height):
             yyy = y * width
             ym1 = ((y + height - 1) % height) * width
             yp1 = ((y + 1) % height) * width
             xm1 = width - 1
             for x in range(width):
                 xp1 = (x + 1) % width
                 neighbors = (
                     old[xm1 + ym1] + old[xm1 + yyy] + old[xm1 + yp1] +
                     old[x   + ym1] +                  old[x   + yp1] +
                     old[xp1 + ym1] + old[xp1 + yyy] + old[xp1 + yp1])
                 new[x+yyy] = neighbors == 3 or (neighbors == 2 and old[x+yyy])
                 xm1 = x

They first setup all the variables they will need for the `x-1`, `x+1`, `y-1`
and `y+1` coordinates, other variations have these calculations inlined into the
neighbour calculation step and I find this way makes it much more clear for
learning.

They then calculate `neighbours`, which is the total number of neighbour cells
that have live cells in them.  They lay this out in a particularily nice way,
almost in the form of a picture:

    neighbors = (
        old[xm1 + ym1] + old[xm1 + yyy] + old[xm1 + yp1] +
        old[x   + ym1] +                  old[x   + yp1] +
        old[xp1 + ym1] + old[xp1 + yyy] + old[xp1 + yp1])


In this formula, picture these as a grid, with the central element missing,
which is the cell we are calculating for.  This formatting doesn't work with
automatic text formatters, but is a really nice teaching tool that really helped
me figure this out finally.

Now, this is just the most naive implementation of the Game of Life, there are
multitudes of ways to speed it up, including new projects that demo it on SIMD
and on the GPU.

I rewrote this in node:

    const width = 40
    const height = 30
    let gen = 0;

    function update(a,b) {
        for (y = 0; y < height; y++) {
            yyy = y * width;
            ym1 = ((y + height - 1) % height) * width
            yp1 = ((y + 1) % height) * width
            xm1 = width - 1
            for (x = 0; x < width; x++) {
                xp1 = (x + 1) % width
                neighbours = (
                    a[xm1 + ym1] + a[xm1 + yyy] + a[xm1 + yp1] +
                        a[x + ym1] + a[x + yp1] +
                        a[xp1 + ym1] + a[xp1 + yyy] + a[xp1 + yp1])
                if (neighbours == 3 || (neighbours == 2 && a[yyy+x])) {
                    b[yyy+x] = 1
                } else {
                    b[yyy+x] = 0
                }
                xm1 = x
            }
        }
    }

    function print(a) {
        console.clear()
        for (y = 0; y < height; y++) {
            yyy = y * width;
            process.stdout.write("|")
            for (x = 0; x < width; x++) {
                c = a[yyy+x] == 0 ? " " : "+"
                process.stdout.write(c)
            }
            process.stdout.write("|\n")
        }
    }

    curr = Array.from({length: width * height},
                      () => Math.floor(Math.random() * 2))
    next = Array.from({length: width * height},
                      () => Math.floor(Math.random() * 2))
    while (true) {
        update(curr,next)
        print(next)
        update(next,curr)
        print(curr)
        gen++
    }

It runs pretty quick on the command line

