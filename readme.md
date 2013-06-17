The goal is simply track whether or not an MLB team plays on any given day
(flagging), and storing this data compactly.

To store a season for a team you might use something like this using JSON:

//  Each team, with an array containing a flag of each day in each month
//  that team plays.
//
{ "teamId": [ "0 : ["0","0","1"..."0","1"], 1 : ["1","1","1"...."1","0"] ... ],
teamId: ... }

Maintaining ~30 x 6 points of data per team, with more to deal with head and
tail if they bleed (ie. If one game in March, you store 30 zeros and a
single one, for example), + space for quotes

You could also do something like this for each month, using strings:

teamId: ["100000111111...1111110001", "11110001....1111111111"....]

And use charAt() or similar.

For 31 you would need 31 bytes.

It wasn't hard to see the bitwise beacon.

A month never has more than 31 days so we know that a 32 bit number safely
contains enough bits to flag calendar data.

Any given team plays almost every day in a month.

Teams play, in the regular season, for about 6 full months, with small
rounding errors at the head and tail.

There are 30 MLB teams.  There are 2430 total games.  Each team plays 162
games.

Let's say a team plays on the 1st, 3-10th, 13th-28th, and the month has 31
days. In bits, beginning rightmost:

0001111111111111111001111111101

Or in decimal:

268432381 -> 9 bytes as a string representation.

(Once converted, storing an integer would also consume less memory than a string, max 4 bytes).

So we could store a month of data in a single number, or several months in
an array of numbers:

{
    teamId: [268432381,234235,1,16,32,64...]
}

To map a calendar view we simply check which bits are set (0-30 of course):

candidate = 268432381;  // Our flags.
do {
    candidate & 1   // if rightmost set, then team plays on current day
} while (candidate >>= 1) // shift right, moving pointer to next day.

To create your map, simply run an or operation against whatever the current
value for a team for a given month is.  From the beginning you would have a
zero(0), and if the team played on the 5th day, then:

// Set bit at position 4 if not already set (OR).
// You'll be running through your game data and doing one for each team,
// for each month, on each day.
//
candidate |= Math.pow(2, 4) // prob. pre-calc powers and put in lookup

I think there are other uses here for calendar based data, esp. if you have
multiple maps (ie. Filters) for any given day.  This might cut down on bytes
over the wire and might result in simple, perhaps compact UI code. Also helps
with localStorage.
