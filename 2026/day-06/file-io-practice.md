<h1>Read and Write Text Files in Linux</h1>

`touch notes.txt `

Create a text file named notes.txt

`echo "Hello" > notes.txt`
Write text to notes.txt

`echo "Linux practice" >> notes.txt`
Append text to notes.txt

`echo "AWS practice" | tee -a notes.txt`
Append text to notes.txt and display it on the terminal

`cat notes.txt`
Read notes.txt

`head -n 2 notes.txt`
Read the first two lines of notes.txt

`tail -n 1 notes.txt`
Read the last line of notes.txt
<img width="1920" height="1080" alt="Screenshot (271)" src="https://github.com/user-attachments/assets/75fcab51-fa57-439a-a398-b20533c1d8c2" />


