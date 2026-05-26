Sprint 3: 13 Mar - 20 Mar

- HLD
  - amplitude system design [x] [x] 
  - New aggregator system design [x] [x] [x]
      `In this round I was asked to design a Google news sort of service where we scrape from multiple news sites. Assume they provide endpoint to get latest 25 news from each publisher. There can be thousands of publishers.
    It was required to write down table schema for all like publisher, user, category_subscription, publisher_subscription, articles etc and handle deduplication in article scraping service etc.
    Along with this, user's should be able to subscribe to certain categories like Education or publishers say ndtv.
    On home page.`
  - Design a system where XXX's customers can upload documents for review by internal audit team. Each document has a time to review complete.
  -  Design hotel reservation aggregator. The only difference was that the hotel can be listed on other aggregators as well and we need to handle this. I moved this requirement to the end and designed rest of the system before
  - 


- LLD:
  - Currency Conversion [x] [x]- 1 SP
  https://leetcode.com/discuss/post/483660/google-phone-currency-conversion-by-anon-upqo/

  - KV Store
  - Delivery Driver - Rate Assignment Problem
    Given a time when we clear payments, output the total that has to be paid. Also, there should be another function that returns the total unpaid.

  - Given a list of songs, and users playing songs, return the list of songs in descending order of most unique users playing the songs.
    Scale up: Return the per user most recently played top 3 songs. I had a discussion about the time complexity of what happens when the number 3 is arbritary and so on. [x] [x] [x]

    Round 1 - DSA
    Design a Music Player like Spotify with below methods

    int addSong(string songTitle); // add a song to your music player with incremental song ids starting from 1
    void playSong(int songId, int userId); // user plays a song that is present in the music player
    void printMostPlayedSongs(); // print song titles in decreasing order of number of unique users' plays
    Follow up

    vector<int> getLastThreeSongs(int userId); // get last 3 unique songs played by a given user

  - The problem was like: given a list of transaction <from, to, amount>, I have to settle them and return who will pay whom, how much. (note: it did not ask me to find the most optimal way to settle like in Splitwise simplify balance feature).

  -  Design an excel sheet

  void set(string cell, string value); // cell can be A1, B2.  value can be like "10", "1" or even excel formulae like "=9+10" and "=-1+-10+2"
  void reset(string cell); // reset the cell 
  void print(); // print all the cells along with their raw and computed values
  Follow up
  Extend solution to support values like "=A1+10" where A1 is a cell name

  - Q. Company xyz.com has an organizational structure such that each employee in the company can have at most one manager
  and may have many subordinates. The company recently conducted their quarterly performance review cycle and each employee has received a performance rating.

  An example structure is as follows:

            A(5)
  B(3)                    C(1)

                  D(4)             E(10)
  A is the manager of B and C
  C is the manager of D and E
  Performance ratings are mentioned in brackets
  Now given the employee information of a company, return the employee whose team has the highest performance rating average.
  A team is defined as a group consisting of an employee and all their subordinates (not just the direct ones).

  Sample input/output:
  Input format: [employee name, rating, List]
  data = [['A', 5, ['B', 'C']], ['B', 3, []], ['C', 2, ['D', 'E']], ['D', 4, []], ['E', 10, []]]
  Output: E

  Modified version of https://leetcode.com/problems/employee-importance/

  -  Implement a port management system

  Assume you are writing the firmware for a network bridge.
  There are N ports each with an unique ID.
  We want to write a class with the get and free methods as shown in the example.
  Let’s start by optimizing for runtime complexity.

  Required methods:

  get() → Return a currently free port.
  Once a port is returned it is considered busy.
  free(port_id) → Frees the given port.
  Nothing should happen if you free a port which is already free.

  -   Optimal acc balancing