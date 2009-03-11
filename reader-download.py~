#!/usr/bin/python

# Copyright 2008 Aron Jones
#
#	This program is free software: you can redistribute it and/or modify
#	it under the terms of the GNU General Public License as published by
#	the Free Software Foundation, either version 3 of the License, or
#	(at your option) any later version.
#
#	This program is distributed in the hope that it will be useful,
#	but WITHOUT ANY WARRANTY; without even the implied warranty of
#	MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#	GNU General Public License for more details.
#
#	You should have received a copy of the GNU General Public License
#	along with this program.  If not, see <http://www.gnu.org/licenses/>.

import urllib, urllib2
import tempfile
import shutil
import getpass
import sys
import os, os.path
import commands
import time
from GoogleReader import reader
from optparse import OptionParser

# edit here to set default values
user = False # Google Reader username and password
password = False
tag = 'hacks' # the tag or label where the torrent RSS feeds are in Google Reader
savedir = os.getcwd() # where the podcasts should be saved. os.getcwd() is the current working dir. no trailing slashes.
prompt = True # whether or not to prompt before downloading
app = False # app with which to open the .torrent

parser = OptionParser(usage='usage: %prog [options]')
parser.add_option("-u", "--user", 
		dest="user", default=user,
		help='Google Reader username. Will prompt if not specified.')
parser.add_option("-p", "--password", 
		dest="password", default=password,
		help='Google Reader password. Will prompt if not specified.')
parser.add_option("-t", "--tag", 
		dest="tag", default=tag,
		help='Tag in Google Reader where the torrents are located. The default is %s.' % tag)
parser.add_option("-l", "--torrent-location", 
		dest="tdir", default=savedir,
		help='Directory where the .torents should be saved (without a trailing slash). The default is current working directory.')
parser.add_option("-f", "--file-location", 
		dest="fdir", default=savedir,
		help='Directory where non .torents files should be saved (without a trailing slash). The default is current working directory.')
parser.add_option("-n", '--no-prompt',
		action="store_false", dest="prompt", default=prompt,
		help='Flag indicating whether or not to prompt before downloading files. The default is %s.' % prompt)
parser.add_option("-a", '--application',
		dest="app", default=app,
		help='If specified, an application will be used to open the .torrent file.')
		
(options, args) = parser.parse_args()

options.user = 'aron.jones'
options.password = 'bokunodaiski'

print 'Running reader-torrent.py at ' + time.strftime('%l:%M:%S %P - %B %e, %G')

#make google reader object
read = reader.GoogleReader()

print 'logging into Google Reader...'

#If there is no need to prompt for u and p, identify now
if options.user and options.password:
	read.identify(options.user, options.password)

if not read.login():
	# prompt for u and p until there is a successful login 
	while True:
		read.identify(raw_input('Google Username:'), getpass.getpass('Password:'))
		if not read.login():
			print 'Invalid Username or Password'
		else:
			break

print 'Checking for new torrents...'

# grab the feed of new torrents. user/-/ indicates the curent logged in user.
# label/tag indicates the tag we want. exclude_target='user/-/state/com.google/read'
# adds ?xt=user/-/state/com.google/read to the url, which indicates we don't want
# read items.
feedurl = 'user/-/label/' + options.tag
feed = read.get_feed(url=feedurl,exclude_target='user/-/state/com.google/read')
print feedurl
print '\nNew Torrents:' # print a list of new podcasts
for entry in feed.get_entries():
	print "	%s" % (entry['title'])
	
	while True: # infinate loop (with manual breaks)
		if options.prompt:
			# ask to go ahead.
			dl_torrent = raw_input('\nDownload? Items will be maked as read in Google Reader. [Y/N]: ')
		elif not options.prompt:
			dl_torrent = 'Y'
		
		if dl_torrent.upper() == 'Y' or 'YES': # if yes
	
			# grabs a Google Reader API token, needed to mark podcasts as read
			read.get_token()
	
			print '\nDownloading ', entry['title']
		
			# open the remote file
			if 'medialink' in entry:
			    remotefile = urllib2.urlopen(entry['medialink'])
			else:
			    remotefile = urllib2.urlopen(entry['link'])
			
			try:
				filename = remotefile.headers['Content-Disposition'].split('"')[-2]
			except:
			    if 'medialink' in entry:
				    filename = entry['medialink']
			    else:
			        filename = entry['link']
			    filename = filename.split('/')[-1].split('#')[0].split('?')[0]
				
			extention = filename.split('.')[-1]
			
			if extention == 'torrent':
				fileloc = options.tdir + '/' + filename
			else:
				fileloc = options.fdir + '/' + filename
			
			try:
				size = int(remotefile.headers['Content-Length'])
			except KeyError:
				size = None
			temp = open(fileloc,"wb")
			fetched = 0
			while 1:
				buf = remotefile.read(1024*20)
				if not buf: break
				temp.write(buf)
				fetched += len(buf)
				if size:
					sys.stdout.write("%.02f%% of %s MB\r" % (100*fetched / float(size), (size/1048576)))
				else:
					sys.stdout.write(" Fetched %d bytes\r" % fetched)
			sys.stdout.flush()
			temp.close()
			remotefile.close()
		
			if options.app and extention == 'torrent':
				# open with app
				print 'opening torrent in ' + options.app
				os.popen(options.app + ' ' + fileloc)
		
			# mark item as read in Google Reader
			read.set_read(entry['google_id'])
			
			print 'Done.'
			break
		elif dl_torrent.upper() == 'N' or 'NO': # if user doesn't want to download the file
			print 'Okay, well, bye then.'
			break
		else: # if input isn't yes or no
			print 'Sorry, what was that?'




