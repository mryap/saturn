#!/usr/bin/env python

import sys
import json
import argparse
import os
import urllib.parse
from os.path import dirname, basename, join, expanduser, exists, abspath
import subprocess


def load_published_data():
    data_path = os.environ.get(
        'PUBLISH_DATA_PATH',
        expanduser("~/.saturn_data/published.json")
    )
    if not exists(dirname(data_path)):
        os.makedirs(dirname(data_path))
    if not exists(data_path):
        return {}
    with open(data_path, "r") as f:
        return json.load(f)


def save_published_data(path, token):
    data_path = os.environ.get(
        'PUBLISH_DATA_PATH',
        expanduser("~/.saturn_data/published.json")
    )
    published_data = load_published_data()
    published_data[path] = token
    with open(data_path, "w+") as f:
        return json.dump(published_data, f)


def load_creds():
    url = os.environ.get('PUBLISH_URL', None)
    token = os.environ.get('SATURN_TOKEN', None)
    if url is None or token is None:
        cpath = expanduser("~/.saturn/config.json")
        if exists(cpath):
            with open(cpath, "r") as f:
                data = json.load(f)
            url = data['url']
            token = data['token']
    return url, token


def execute(cmd):
    print(" ".join(cmd))
    proc = subprocess.Popen(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    proc.wait()
    stdout, stderr = proc.communicate()
    code = proc.returncode
    return code, stdout.decode('utf-8'), stderr.decode('utf-8')

def run(path):
    published_data = load_published_data()
    url, token = load_creds()
    if path[-1] == '/':
        path = path[:-1]
    existing_token = published_data.get(path, None)
    tarpath = join('/tmp', "%s.tar.gz" % basename(path))
    dirpath = dirname(path)
    if not dirpath:
        dirpath = "."
    cmd=["tar",
         "--exclude",
         "*.git*",
         "-cvzf",
         tarpath,
         "-C",
         dirpath,
         basename(path)]
    code, stdout, stderr = execute(cmd)
    if stdout:
        print(stdout)
    if stderr:
        print(stderr)
    if code != 0:
        raise SystemExit()
    dest = "%s/%s" % (url, urllib.parse.quote(basename(tarpath)))
    if existing_token:
        dest += "?token=%s" % existing_token
    cmd = [
        "curl",
        "-X",
        "PUT",
        "-u",
        "%s:" % token,
        "-T",
        tarpath,
        dest
    ]
    code, stdout, stderr = execute(cmd)
    if stderr:
        print('stderr', stderr)
    try:
        data = json.loads(stdout)
        if data['status'] == 'error':
            print(data['message'])
            print('There has been an unknown error.  please contact support@saturncloud.io')
        token = data['token']
        save_published_data(abspath(path), token)
    except Exception as e:
        print(stdout)
    print ("Please visit the publishing section on your Saturn Dashboard to finish publication")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument('path')
    args = parser.parse_args()
    path = args.path
    run(path)
