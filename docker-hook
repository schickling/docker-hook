#! /usr/bin/env python
# -*- coding: utf-8 -*-

"""Automatic Docker Deployment via Webhooks"""

import json
import os
from subprocess import Popen
from argparse import ArgumentParser, ArgumentDefaultsHelpFormatter
try:
    # For Python 3.0 and later
    from http.server import HTTPServer
    from http.server import BaseHTTPRequestHandler
except ImportError:
    # Fall back to Python 2
    from BaseHTTPServer import BaseHTTPRequestHandler
    from BaseHTTPServer import HTTPServer as HTTPServer
import sys
import logging
import requests

logging.basicConfig(format='%(asctime)s %(levelname)s %(message)s',
                    level=logging.DEBUG,
                    stream=sys.stdout)


class RequestHandler(BaseHTTPRequestHandler):
    """A POST request handler which expects a token in its path."""
    def do_POST(self):
        logging.info("Path: %s", self.path)
        header_length = int(self.headers.getheader('content-length', "0"))
        json_payload = self.rfile.read(header_length)
        env = dict(os.environ)
        json_params = {}
        if len(json_payload) > 0:
            json_params = json.loads(json_payload)
            env.update(('REPOSITORY_' + var.upper(), str(val))
                       for var, val in json_params['repository'].items())

        # Check if the secret URL was called
        if args.token in self.path:
            logging.info("Start executing '%s'" % args.cmd)
            try:
                Popen(args.cmd, env=env).wait()
                self.send_response(200, "OK")
                if 'callback_url' in json_params:
                    # Make a callback to Docker Hub
                    data = {'state': 'success'}
                    headers = {'Content-type': 'application/json',
                               'Accept': 'text/plain'}
                    requests.post(json_params['callback_url'],
                                  data=json.dumps(data),
                                  headers=headers)
            except OSError as err:
                self.send_response(500, "OSError")
                logging.error("You probably didn't use 'sh ./script.sh'.")
                logging.error(err)
                if 'callback_url' in json_params:
                    # Make a callback to Docker Hub
                    data = {'state': 'failure',
                            'description': str(err)}
                    headers = {'Content-type': 'application/json',
                               'Accept': 'text/plain'}
                    requests.post(json_params['callback_url'],
                                  data=json.dumps(data),
                                  headers=headers)
        else:
            self.send_response(401, "Not authorized")
        self.end_headers()


def get_parser():
    """Get a command line parser for docker-hook."""
    parser = ArgumentParser(description=__doc__,
                            formatter_class=ArgumentDefaultsHelpFormatter)

    parser.add_argument("-t", "--token",
                        dest="token",
                        required=True,
                        help=("Secure auth token (can be choosen "
                              "arbitrarily)"))
    parser.add_argument("-c", "--cmd",
                        dest="cmd",
                        required=True,
                        nargs="*",
                        help="Command to execute when triggered")
    parser.add_argument("--addr",
                        dest="addr",
                        default="0.0.0.0",
                        help="address where it listens")
    parser.add_argument("--port",
                        dest="port",
                        type=int,
                        default=8555,
                        metavar="PORT",
                        help="port where it listens")
    return parser


def main(addr, port):
    """Start a HTTPServer which waits for requests."""
    httpd = HTTPServer((addr, port), RequestHandler)
    httpd.serve_forever()

if __name__ == '__main__':
    parser = get_parser()
    if len(sys.argv) == 1:
        parser.print_help()
        sys.exit(1)
    args = parser.parse_args()
    main(args.addr, args.port)
