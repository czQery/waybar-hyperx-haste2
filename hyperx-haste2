#!/bin/bash

HID_ID="000003F0:00000F98"
HID_INPUT="input2"
HID_OFFSET=66
HID_LENGTH=2
HID_REQUEST="\x50\x02\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00"
HID_RESPONSE="5102"
HID_INDEX=-1

PIPE=/tmp/hyperx-haste2
CACHE=~/.cache/hyperx-haste2

set -o nounset
shopt -s nullglob

if [ ! -e $PIPE ]; then
	mkfifo $PIPE
fi

if [ ! -e $CACHE ]; then
	touch $CACHE
fi

result() {
	if [[ $1 > 0 ]]; then
		echo "$(printf "%d" "0x$1")%" > $CACHE
	fi

	if [ ! -s $CACHE ]; then
		exit 1
	fi

	cat "$CACHE"
	exit 0
}

for dev in /sys/class/hidraw/hidraw*; do
	uevent="$dev/device/uevent"
	[ -r "$uevent" ] || continue

	id=$(grep -m1 '^HID_ID=' "$uevent" || true)
	phys=$(grep -m1 '^HID_PHYS=' "$uevent" || true)
	[ -n "$id" ] || continue
	[ -n "$phys" ] || continue

	if [[ "$id" == *"$HID_ID"* && "$phys" == *"$HID_INPUT"* ]]; then
		HID_INDEX=${dev##*/hidraw}
		break
	fi
done

if [[ $HID_INDEX == -1 ]]; then # device not found
	result 0
fi

timeout 3 head -c 128 "/dev/hidraw$HID_INDEX" | hexdump -v -e '1/1 "%02x"' > $PIPE &
printf "$HID_REQUEST" > "/dev/hidraw$HID_INDEX"

response=$(cat $PIPE)
if [[ "$response" != *"$HID_RESPONSE"* ]]; then # invalid response
	result 0
fi

value=$(echo "$response" | cut -c $(($HID_OFFSET * 2 + 1))-$(($HID_OFFSET * 2 + $HID_LENGTH)))
if [ -z "$value" ]; then # invalid response value
	result 0
fi

result $value
