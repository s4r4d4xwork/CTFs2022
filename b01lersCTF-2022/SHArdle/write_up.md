![image](https://user-images.githubusercontent.com/101840614/165021972-42f637ad-5254-4672-a435-1cdb57777c6d.png)

I think this challenge is same like this chall [passWORDLE](https://rsk0315.github.io/playground/passwordle.html)

# Meaning it changes our input to hash SHA256 and compare with the key 'guess' hash and turn the right postion and value to GREEN word color, the right value but not in right position to YELLOW word color , and the rest to GRAY word color

Given chall code(SHArdle.py):
```python
from SHArdle_helpers import FLAG, generate, valid

from Crypto.Hash import SHA256
from Crypto.Random.random import randint
from collections import Counter


def hash(s):
   h = SHA256.new()
   h.update(s)
   return h.hexdigest()

def green(c):   return '\033[42m' + c + '\033[0m'
def gray(c):    return '\033[40m' + c + '\033[0m'
def yellow(c):  return '\033[43m' + c + '\033[0m'

def compare(s, s0):
   n = len(s0)
   assert len(s) == n
   # get greens
   ans = [None]*n
   rest0 = Counter()
   for i in range(n):
      if s[i] == s0[i]: ans[i] = green(s[i])
      else: rest0.update(s0[i])
   # color rest
   for i in range(n):
      if ans[i] != None: continue
      cnt = rest0.get(s[i])
      if cnt == None or cnt == 0: ans[i] = gray(s[i])
      else:
         rest0[s[i]] -= 1
         ans[i] = yellow(s[i])
   return "".join(ans)


def hashAndCompare(s, s0):
   h = hash(s)
   h0 = hash(s0)
   return compare(h, h0)


## GAME


class Game:

   def __init__(self):
      self.rounds = 2
      self.secrets = [ generate(randint)   for i in range(self.rounds) ]
      self.start()

   def menu(self, round):
      print(f"1 - guess secret {round}")
      print( "2 - exit")
      print( "choose: ", end = "")

   def start(self):

      print( "I thought of some secrets, can you guess those?\n" )

      guesses = 15
      round = 0
      while guesses > 0:
         try:
            self.menu(round + 1)
            s = int( input("").strip() )
            if s == 2:  exit(0)
            elif s == 1:
               s = input("your guess: ").strip().encode().lower()
               if not valid(s):  print("INVALID GUESS")
               else:
                  print(f"score: {hashAndCompare(s, self.secrets[round])}")
                  if s == self.secrets[round]:
                     round += 1
                     print(f"BINGO!! {self.rounds - round} more to go")
                     if round == self.rounds:
                        print(f"Here is the flag: {FLAG}")
                        exit(0)
                  guesses -= 1
         except ValueError:
            print("invalid choice")
      print("No more guesses - goodbye!")


if __name__ == "__main__":

   game = Game()


```

# After few tries, I realise the valid() function only accept meaning ENGLISH words ([English-dictionary wordlist](https://www.wordgamedictionary.com/sowpods/download/sowpods.txt))

# I wrote a py code to get request and recive from sever (same like thing you do on terminal) to get the GREEN position:
```python
import socket
from textwrap import wrap



def main():
    final_arr = ['*']*64
    round = 0
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("ctf.b01lers.com", 9102))
    print('Recv: \n' + s.recv(1024).decode('utf-8'))

    for j in range(15):
        if round == 1:
            final_arr = ['*']*64
        n = '1' + '\r\n'
        s.send(n.encode())
        print('Recv: \n' + s.recv(1024).decode('utf-8'))

        result_str = input(f"GUESS {j} :").strip().lower() + '\r\n'
        s.send(result_str.encode())
        temp = s.recv(1024)
        print(result_str)
        print('Recv: \n' + temp.decode('utf-8'))

        str_recv = str(temp)
        if 'BINGO!!' in str_recv:
            print('---BINGO!!!---')
            round = round + 1
            print('ROUND2!!!' + str(round))
        str_recv = str_recv[9:]

        if True:
            if str_recv.startswith(r"\x1b"):
                result_arr = wrap(str_recv, 16)
                result_arr = result_arr[:64]
                i = 0

                for item_arr in result_arr:

                    if item_arr.startswith(r'\x1b[42m'):
                        #
                        final_arr[i] = item_arr[8:9]
                    i = i + 1
                print(final_arr)
                for item in range(64):
                    if final_arr[item] != '*':
                        print(f'Value: {final_arr[item]} - Index: {item}')
            else:
                continue

    s.close()


main()

```

# => So I wrote a python code to get the hash that have exact GREEN position in output of above code, and find the exact key 'guess' word :

```python
import hashlib
from collections import Counter

possible_array = []

hash_arr = [] #green inputs
def count_letters(letters, characters):
    counters = Counter(letters)
    return counters[characters]

def comparision2(arr1, str):
    arr2 =list(str)
    
    for i in range(64):
        if arr1[i] != '*':
            if arr1[i] != arr2[i]:
                return False
    return True

def comparison(key, guess):
    keyc = Counter(key)
    guessc = Counter(guess)
    hexa = '0123456789abcdef'
    for item in hexa:
        if guessc[item] < keyc[item]:
            return False
    return True

            

with open('words.txt', "r") as f:
    txt = f.read()
txt_array = txt.split("\n")

hashed_array = ['*'] * len(txt_array)

#key = hashlib.sha256('flag'.encode('utf-8')).hexdigest()
count = 0
for item in txt_array:
    hashed_value = hashlib.sha256(item.encode('utf-8')).hexdigest()
    if comparision2(hash_arr, hashed_value):
        print(f'\nGOT IT! - Index: {txt_array.index(item)} - Value: {item} - HashedValue: {hashed_value}\n')
    # if comparison(key,hashed_value) and item.isalpha():
    #     if comparision2(hash_arr,hashed_value):
    #         print(f'\nGOT IT! - Index: {txt_array.index(item)} - Value: {item} - HashedValue: {hashed_value}\n')
print('---DONE!---')

```

# After 2 round, you will get the flag! GOOD LUCK (the sever is closed so I can't post detail image of my above codes do, sorry :<)
![image](https://user-images.githubusercontent.com/101840614/165023445-bc7ec1cf-2b91-4bfa-814b-1ea4b417ed43.png)
