#!/usr/bin/env python

from __future__ import print_function

import roslib; roslib.load_manifest('teleop_twist_keyboard')
import rospy

from geometry_msgs.msg import Twist

import sys, select, termios, tty

# Twitch Stuff here
import irc.bot
import requests


username  = "twitch-plays-teleop-bot"
client_id = "hnisz359piwaj7htdcup5j32lhdhgz"
token     = "vpw61tcfczvwaytdr6xiwetdf1yvhy"
channel   = "fasermaler"


msg = """
Reading from Twitch and Publishing to Twist!
---------------------------
Moving around:
    up - move up
    down - move down
    left - move left
    right - move right
    stop - stop

CTRL-C to quit
"""



def vels(speed,turn):
    return "currently:\tspeed %s\tturn %s " % (speed,turn)
def publish_teleop(x, y, z, th):


    twist = Twist()
    twist.linear.x = x*speed; twist.linear.y = y*speed; twist.linear.z = z*speed;
    twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = th*turn
    pub.publish(twist)

class TwitchBot(irc.bot.SingleServerIRCBot):
    def __init__(self, username, client_id, token, channel):
        self.client_id = client_id
        self.token = token
        self.channel = '#' + channel

        # Get the channel id, we will need this for v5 API calls
        url = 'https://api.twitch.tv/kraken/users?login=' + channel
        headers = {'Client-ID': client_id, 'Accept': 'application/vnd.twitchtv.v5+json'}
        r = requests.get(url, headers=headers).json()
        self.channel_id = r['users'][0]['_id']

        # Create IRC bot connection
        server = 'irc.chat.twitch.tv'
        port = 6667
        print('Connecting to ' + server + ' on port ' + str(port) + '...')
        irc.bot.SingleServerIRCBot.__init__(self, [(server, port, 'oauth:'+token)], username, username)
        

    def on_welcome(self, c, e):
        print('Joining ' + self.channel)

        # You must request specific capabilities before you can use them
        c.cap('REQ', ':twitch.tv/membership')
        c.cap('REQ', ':twitch.tv/tags')
        c.cap('REQ', ':twitch.tv/commands')
        c.join(self.channel)


    def on_pubmsg(self, c, e):
        chat_input = e.arguments[0]
        # If a chat message starts with an exclamation point, try to run it as a command
        if 'up' in chat_input.lower():
            print('Bot going up!')
            publish_teleop(1, 0, 0, 0)
            

        elif 'down' in chat_input.lower():
            print('Bot going down!')
            publish_teleop(-1, 0, 0, 0)

        elif 'left' in chat_input.lower():
            print('Bot going left!')
            publish_teleop(0, 0, 0, 20)

        elif 'right' in chat_input.lower():
            print('Bot going right!')
            publish_teleop(0, 0, 0, -20)


        elif 'stop' in chat_input.lower():
            print('Bot is stopping!')
            publish_teleop(0, 0, 0, 0)

        elif 'help' in chat_input.lower():
            self.print_help(c)


    def print_help(self, c):
        url = 'https://api.twitch.tv/kraken/channels/' + self.channel_id
        headers = {'Client-ID': self.client_id, 'Accept': 'application/vnd.twitchtv.v5+json'}
        r = requests.get(url, headers=headers).json()
        c.privmsg(self.channel, 'This is twitch plays teleop bot!')
        c.privmsg(self.channel, 'Commands:')
        c.privmsg(self.channel, 'up - bot goes up')
        c.privmsg(self.channel, 'down - bot goes down')
        c.privmsg(self.channel, 'left - bot turns left')
        c.privmsg(self.channel, 'right - bot turns right')
        c.privmsg(self.channel, 'stop - bot stops')

    


def main():
    print(msg)
    print(vels(speed,turn))

    bot = TwitchBot(username, client_id, token, channel)
    bot.start()

if __name__ == "__main__":
    settings = termios.tcgetattr(sys.stdin)

    pub = rospy.Publisher('cmd_vel', Twist, queue_size = 1)
    rospy.init_node('teleop_twist_twitch')

    speed = rospy.get_param("~speed", 0.5)
    turn = rospy.get_param("~turn", 1.0)
    x = 0
    y = 0
    z = 0
    th = 0
    status = 0
    try:
        main()
    except Exception as e:
        print(e)

    finally:
        twist = Twist()
        twist.linear.x = 0; twist.linear.y = 0; twist.linear.z = 0
        twist.angular.x = 0; twist.angular.y = 0; twist.angular.z = 0
        pub.publish(twist)

        termios.tcsetattr(sys.stdin, termios.TCSADRAIN, settings)







            

    
