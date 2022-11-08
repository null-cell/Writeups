# Hambone challenge writeup

![Screenshot of the challenge](https://cdn.discordapp.com/attachments/671823650491465748/1038966502826717224/chall_pic.png)
 Once we open the site, we are met with a **This is not the right way** sign. We also got a file that comes with the challenge called dist.py: 
 

    def get_distances(padded_url : str, flag_path : str):
    distances = []
    for i in range(3):
        # calculate hamming distance on 16 byte subgroups
        flag_subgroup = flag_path[i*32:i*32+32]

        z = int(padded_url[i*32:i*32+32], 16)^int(flag_subgroup, 16)
        distances.append(bin(z).count('1'))  
        
    return distances


 From the code and challenge description we can infer, that the background color is just a hex of three hemming distances of the 16 byte subgroups and our guess (padded hex numbers).

Now we just need to find out which 48 byte value xor's with the flag path to produce #fffff (since we got a hint that #000000 is not the right way). To do this, we can check each **bit** separately - to guess the bit at i-th position, we check a string of only 0's and a 1 at that bit. Starting with the least significant bit position, we take 1, pad it to 32 characters (16 bytes) and send it to the url. Then we check which scores lowered, meaning that there is a 1 in the flag_path at that position for that part of the flag. Then we shift the bit to the left and repeat this, until we have checked all bits. 

    i=1
    while i < 2**128:
	    start = str(hex(i)[2:])
	    start = start.zfill(32)
	    
	    r = requests.get(url+start*3)

	    background_color = r.text.split('background: #')[1].split('"')[0]
	    n1 = int(background_color[:2], 16)
	    n2 = int(background_color[2:4], 16)
	    n3 = int(background_color[4:6], 16)

	    
	    if n1 < c1:
	        firstflag += "1"
	    else:
	        firstflag += "0"
	    if n2 < c2:
	        secondflag += "1"
	    else:
	        secondflag += "0"
	    if n3 < c3:
	        thirdflag += "1"
	    else:
	        thirdflag += "0"

	    i = i << 1

 
Once this is done, we just reverse the flag parts, since we were adding them in the opposite order. Because we want the highest amount of bits possible, we invert the bits to reveal where there were no 1's. Then we just convert the flag parts to hex and concatenate them.

    firstflag = int(firstflag[::-1], 2) ^ (2**128-1)
    secondflag = int(secondflag[::-1], 2) ^ (2**128-1)
    thirdflag = int(thirdflag[::-1], 2) ^ (2**128-1)

    r = requests.get(url+str(hex(firstflag))[2:]+str(hex(secondflag))[2:]+str(hex(thirdflag))[2:])
    background_color = r.text.split('background: #')[1].split('"')[0]
    
    print(hex(int(background_color, 16)))
    print(r.text)

Below you can find the whole solution script:

    
    import requests
    
    url = "https://hambone.chall.pwnoh.io/"
    
    c1 = int("c6", 16)
    c2 = int("b4", 16)
    c3 = int("c5", 16)
    
    firstflag= ""
    secondflag=""
    thirdflag=""
    i=1
    
    while i < 2**128:
	
	    start = str(hex(i)[2:])
	    start = start.zfill(32)

	    r = requests.get(url+start*3)
	    background_color = r.text.split('background: #')[1].split('"')[0]

	    n1 = background_color[:2]
	    n2 = background_color[2:4]
	    n3 = background_color[4:6]

	    n1 = int(n1, 16)
	    n2 = int(n2, 16)
	    n3 = int(n3, 16)

	    if n1 < c1:
	        firstflag += "1"
	    else:
	        firstflag += "0"
	    if n2 < c2:
	        secondflag += "1"
	    else:
	        secondflag += "0"
	    if n3 < c3:
	        thirdflag += "1"
	    else:
	        thirdflag += "0"

	    i = i << 1


    firstflag = firstflag[::-1]
    secondflag = secondflag[::-1]
    thirdflag = thirdflag[::-1]
    
    firstflag = int(firstflag, 2) ^ (2**128-1)
    secondflag = int(secondflag, 2) ^ (2**128-1)
    thirdflag = int(thirdflag, 2) ^ (2**128-1)
        
    r = requests.get(url+str(hex(firstflag))[2:]+str(hex(secondflag))[2:]+str(hex(thirdflag))[2:])
    background_color = r.text.split('background: #')[1].split('"')[0]
    
    print(hex(int(background_color, 16)))
    
    print(r.text)

