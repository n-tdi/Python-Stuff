/**
 * The Javascript logic for Toolkit that runs on the client's browser.
 * TODO: If you're anyone but Jeff - I would avoid working in this file if you value your sanity.
 * TODO: It's an encapsulated legacy file for a reason.
 */

"use strict";

/**
 * The Connection class. Responsible for connection between local and remote servers.
 * @type {Connection}
 */
let Connection = (function ()
{
    "use strict";

    function Connection () {}

    /**
     * Connection properties.
     * @lends Connection.prototype
     */
    let connectionProperties = {
        ws: {
            enumerable: true,
            get: function () {
                return this._ws;
            }
        },
        ready: {
            enumerable: true,
            get: function () {
                return this._ready;
            }
        },
        connecting: {
            enumerable: true,
            get: function () {
                return this._connecting;
            }
        },
        _ws: {
            writable: true
        },
        _ready: {
            writable: true,
            value: false
        },
        _connecting: {
            writable: true,
            value: false
        },
        _listeners: {
            writable: true,
            value: {}
        },
        _callbacks: {
            writable: true,
            value: {}
        },
        _errorCallbacks: {
            writable: true,
            value: {}
        },
        _messageCallbacks: {
            writable: true,
            value: {}
        },
        _lastId: {
            writable: true,
            value: 0
        }
    };
    Object.defineProperties(Connection.prototype, connectionProperties);

    /**
     * err should only be populated in the protocol if something unexpected happens.
     * If it is simply the server not running at the given port and host, that shouldn't
     * trigger an error, just connection = false.
     *
     * @param {{port:number?, protocol:string?, host}} options
     */
    Connection.prototype.connect = function (options)
    {
        this._connecting = true;

        options = options || {};

        let port = options.port;
        let protocol = options.protocol;
        let host = options.host || "ws://localhost";
        let eventHandler = (function (self) {
            return function (e) {
                handleEvent.call(self, e);
            }
        })(this);

        if (!port || !protocol) {
            throw new Error('Port and protocol must be provided must be provided.');
        }

        this._ws = new WebSocket(host + ":" + port, protocol);

        // onopen and onerror call our callback as appropriate and replace themselves with the proper callback afterwards.
        this._ws.onopen = eventHandler;
        this._ws.onerror = eventHandler;
        this._ws.onmessage = eventHandler;
        this._ws.onclose = eventHandler;

        let connectionOpenListener = function () {
            this._ready = true;
            this._connecting = false;
            this.removeListener(connectionOpenListener);
            this.removeListener(connectionErrorListener);
        };

        let connectionErrorListener = function () {
            this._connecting = false;
            this.removeListener(connectionOpenListener);
            this.removeListener(connectionErrorListener);
        };

        this.addListener('open', connectionOpenListener);
        this.addListener('error', connectionErrorListener);

        this.addListener('message', function (e) {
            try {
                let parsed = JSON.parse(e.data);

                if (parsed.id) {
                    if (parsed.type === 'response') {
                        this._callbacks[parsed.id].apply(this, parsed.data);
                    } else if (parsed.type === 'error') {
                        this._errorCallbacks[parsed.id].apply(this, parsed.data);
                    }

                    if (parsed.type === 'response' || parsed.type === 'error') {
                        delete this._callbacks[parsed.id];
                        delete this._errorCallbacks[parsed.id];
                    }
                }
            } catch (e) {}
        });

        this.addListener('close',function () {
            this._ready = false;
        });
    };

    /**
     *
     * @param message
     */
    Connection.prototype.send = function (message)
    {
        this._ws.send(message);
    };

    /**
     * @param {string} command The command (and namespace if other than default).
     * @param parameters
     * @param {function (err:string, response:*)} callback
     * @param {function (err:string, response:*)} error_callback
     */
    Connection.prototype.sendCommand = function (command, parameters, callback, error_callback)
    {
        let parts = command.split(/\./g);
        command = parts[parts.length-1];
        let namespace = parts.length >= 2 ? parts.slice(0, parts.length-1).join('.') : 'default';

        this._ws.send(JSON.stringify({id:this._getNextId(), command:command, namespace:namespace, parameters:parameters}));
        this._callbacks[this._lastId] = callback || function () {};
        this._errorCallbacks[this._lastId] = error_callback || function () {};
    };

    /**
     *
     * @returns {Connection._lastId|{writable, value}|*}
     * @private
     */
    Connection.prototype._getNextId = function ()
    {
        this._lastId = this._lastId + 1;
        return this._lastId;
    };

    /**
     *
     * @param event
     * @param listener
     */
    Connection.prototype.addListener = function (event, listener)
    {
        if (!this._listeners[event]) {
            this._listeners[event] = [];
        }

        this._listeners[event].push(listener);
    };

    /**
     *
     * @param event
     * @param listener
     */
    Connection.prototype.removeListener = function (event, listener)
    {
        if (!event) { // remove all listeners
            this._listeners = {};
        } else if (!listener) { // remove all listeners for a single event
            this._listeners[event] = [];
        } else if (this._listeners[event]) {
            let listeners = this._listeners[event];
            for (let i = 0; i < listeners.length; i++) {
                if (listeners[i] === listener) {
                    listeners.splice(i, 1);
                    i--;
                }
            }
        }
    };

    /**
     * @this {Connection}
     * @param e
     */
    function handleEvent (e)
    {
        if (this._listeners[e.type]) {
            this._listeners[e.type].forEach(function (listener) {
                listener.call(this, e);
            }, this);
        }
    }

    return Connection;
})();

/**
 * GamePlan Course class to match the server-side values for auto-complete. Excludes unused parameters.
 */
let Course = (function() {
    /**
     * @constructor
     */
    function Course(data)
    {
        this.divisionID = data['divisionID'];
        this.divisionName = data['divisionName'];
        this.divisionShortName = data['divisionShortName'];
        this.divisionSortOrder = data['divisionSortOrder'];
        this.locationID = data['locationID'];
        this.locationName = data['locationName'];
        this.locationShortName = data['locationShortName'];
        this.productBackupFlags = data['productBackupFlags'];
        this.productBackupFolder = data['productBackupFolder'];
        this.productID = data['productID'];
        this.productName = data['productName'];
        this.productShortName = data['productShortName'];
        this.productURLStaff = data['productURLStaff'];
        this.productURLStudent = data['productURLStudent'];
        this.sessionEndDate = data['sessionEndDate'];
        this.sessionID = data['sessionID'];
        this.sessionStartDate = data['sessionStartDate'];
    }

    return Course;
}());

/**
 * Class for handling localStorage data for a specific User.
 */
let LocalUserData = (function()
{
    /**
     * Constants.
     * Note: Status codes should coincide with Toolkit Status CSS class names.
     */
    LocalUserData.STATUS_DISABLED = 'disabled';
    LocalUserData.STATUS_CONNECTED = 'connected';
    LocalUserData.STATUS_CONNECTING = 'connecting';
    LocalUserData.STATUS_ERROR = 'error';
    LocalUserData.CHECK_IN_MINUTE_THRESHOLD = 60;
    LocalUserData.KEY_PREFIX = 'gp/toolkit/user/'; // Combined with Pulse User ID (i.e. 'gp/toolkit/user/123456')

    /**
     * @param {Object} userDataJson
     * @constructor
     */
    function LocalUserData(userDataJson)
    {
        /** @type int */
        this.userID = userDataJson['userID'];
        /** @type string */
        this.studentName = userDataJson['studentName'];
        /** @type Course */
        this.courseInSession = new Course(userDataJson['courseInSession']);
        /** @type string */
        //this.evalUrl = userDataJson['evalUrl'];
        // LocalStorage is never cleared manually - this could allow someone to fill out another Student's Eval

        /** @type Date */
        this.nowDate = new Date();
        /** @type Date */
        this.lastCheckDate = null; // Date Object - Timestamp for last check-in for the current Session.
        /** @type Date */
        this.lastLoginDate = new Date(userDataJson['lastLogin']['date']); // Date Object - Timestamp for last login for the current Session.
        /** @type string */
        this.status = LocalUserData.STATUS_DISABLED; // String - Toolkit Status. Visual Status notifications for TK.

        this.load(); // Load the previous LocalUserData from localStorage (if any) and compare to this new set of data.
    }

    /**
     * Determines whether or not we should attempt to connect to Toolkit based on a set of pre-conditions.
     */
    LocalUserData.prototype.shouldAttemptConnection = function()
    {
        console.log('Testing if Toolkit should attempt connection for Course ID: ', this.courseInSession.productID);
        if (!this.courseInSession.productID) { // if the server determines that the current User does *not* have an active Course ID
            this.setStatus(LocalUserData.STATUS_DISABLED); // Override all other considerations and disable immediately
            return false;
        }

        let checkMinutes = Math.round((this.nowDate.getTime() - this.lastCheckDate.getTime()) / 1000 / 60);
        let loginMinutes = Math.round((this.nowDate.getTime() - this.lastLoginDate.getTime()) / 1000 / 60);

        console.log('Previous Status: ' + this.status);
        console.log('Last Check Timestamp: ' + this.lastCheckDate);
        console.log('Last Login Timestamp: ' + this.lastLoginDate);
        console.log('Now Timestamp: ' + this.nowDate);
        console.log('~' + checkMinutes + ' minutes have passed since last check-in for this Session.');
        console.log('~' + loginMinutes + ' minutes have passed since the User last logged in.');
        console.log('Check-in minute threshold: ' + LocalUserData.CHECK_IN_MINUTE_THRESHOLD + ' minutes.');

        if (this.lastCheckDate === this.nowDate || // User data initialized just now
            this.status === LocalUserData.STATUS_ERROR || // Page refresh + last known status was an error
            loginMinutes <= 1 || // User just logged in (1 minute padding for server script to client script execution)
            checkMinutes >= LocalUserData.CHECK_IN_MINUTE_THRESHOLD) { // User hasn't checked in for a period of time
            console.log('Toolkit check-in parameter thresholds reached. Running Toolkit Web commands.');
            this.lastCheckDate = new Date(this.nowDate.getTime());
            this.save();
            return true;
        }

        console.log('Toolkit check-in parameter thresholds not reached. Bypassing Toolkit Web commands.');
        return false;
    };

    /**
     * Save User data as stringified JSON to localStorage.
     */
    LocalUserData.prototype.save = function()
    {
        let key = LocalUserData.KEY_PREFIX + this.userID;

        localStorage.setItem(key, JSON.stringify(this));

        console.log('Updated local Toolkit User data for key: ' + key);
        console.log('data:', JSON.parse(localStorage.getItem(key)));
    };

    /**
     * Attempt to Load User data from localStorage if it exists. Save a small subset of data to the current iteration
     * based on the contents. Otherwise initialize it.
     */
    LocalUserData.prototype.load = function()
    {
        let key = LocalUserData.KEY_PREFIX + this.userID;

        if (localStorage.getItem(key)) {
            console.log('Found local Toolkit User data for key: ' + key);
            /** @type LocalUserData prevLocalUserData */
            let prevLocalUserData = JSON.parse(localStorage.getItem(key));
            console.log('Previous Local User Data:', prevLocalUserData);
            if (prevLocalUserData.lastCheckDate && prevLocalUserData.lastLoginDate && prevLocalUserData.status) {
                this.lastCheckDate = new Date(prevLocalUserData.lastCheckDate);
                let prevLoginDate = new Date(prevLocalUserData.lastLoginDate);
                this.status = prevLocalUserData.status;
                // Compare the last reported login from server to the one saved in localStorage. If different, update data and save immediately.
                if (this.lastLoginDate.getTime() !== prevLoginDate.getTime()) { // i.e. User just logged in.
                    this.save(); // Save the new lastLoginDate to localStorage for the next check-in.
                }
                return;
            } else {
                console.log('Invalid local User data detected. Re-initializing values.');
            }
        }

        // Initialize from scratch.
        console.log('Initializing local Toolkit User data for key: ' + key);
        // localStorage.clear(); // As long as there is no sensitive data stored - keep alive for debug purposes.
        this.lastCheckDate = this.nowDate;
        this.status = LocalUserData.STATUS_DISABLED;
        this.save();
    };

    /**
     *
     */
    LocalUserData.prototype.getStatus = function()
    {
        return this.status;
    };

    /**
     *
     * @param status
     */
    LocalUserData.prototype.setStatus = function(status)
    {
        this.status = status;
        this.save();
    };

    return LocalUserData;
}());

/**
 * Main Toolkit logic.
 */
(function () {
    // Embedded Configuration Variables
    const port = 12345;
    const protocol = 'id-toolkit-protocol'; // TODO: Need Windows manipulation for this to work.
    const failedAttemptLimit = 5;
    const connectionInterval = 50; // in milliseconds
    const uiFeedbackDelayShort = 500; // Short delay for async operations. Mainly for User experience/feedback registration.
    const uiFeedbackDelay = 1500; // Longer delay for async operations.
    const debug = $('.sf-toolbar').length > 0; // Simple check for existence of Symfony Toolbar. If exists, Toolkit debug mode.

    let ctrlDown = false; // Command (for Mac) and Control (for Win/Linux)

    console.log = console.log || function () {}; // prevent undefined console.log errors.

    let connection;
    //let webService;
    //let $ = jQuery; // Assumes jQuery is available and processed BEFORE this file.
    let failedAttempts = 0;
    let connectionAttempted = false;
    let connectionIntervalId;
    let $navbar = $('body.idtech.gameplan nav #bar'); // Classes to match exact GP Twig settings.
    if (!$navbar.length) {
        return;
    }

    let $toolkitStatus = $navbar.find('#toolkit-status'); // VERY specific to avoid conflicts with 3rd party content.
    let $toolkitModal = $navbar.find('#toolkit-modal');
    if (!$toolkitStatus.length || !$toolkitModal.length) {
        return;
    }

    // Check if there is a Course currently in Session for the current User (Student only) - if so, proceed.
    // In every other case, Toolkit is unnecessary.
    /** @type string */
    let userDataString = $toolkitStatus.attr('data-student');
    if (!userDataString) {
        console.log('Valid Toolkit page, but no Toolkit data is set for this Session. Bypassing Toolkit.');
        return;
    }

    // A Course is in Session at camp for the current User. Retrieve it and decode it.
    /** @type Object */
    let userDataJson;
    try {
        userDataJson = JSON.parse(userDataString);
        if (!userDataJson || !userDataJson['userID'] || !userDataJson['studentName'] || !userDataJson['courseInSession']) {
            throw new Error('Invalid or missing critical Toolkit data.');
        }
    } catch (error) { // SyntaxError
        console.log('Warning: Invalid Toolkit data.');
        console.log('Given string: ', userDataString);
        console.log(error);
        return;
    }

    // Save data specific to the current User via localStorage alongside other Users on this browser (for debug).
    // There should be no sensitive data stored in these values at any time - only used for Toolkit commands.
    let localUserData = new LocalUserData(userDataJson);

    let directories;
    if (!localUserData.courseInSession.productBackupFolder) {
        directories = ['%DESKTOP%/%STUDENT%']; // Fallback in case Pulse field for backup directories is not filled out.
    } else {
        directories = localUserData.courseInSession.productBackupFolder.split('|');
    }

    let osDirectories; // Localized directories per OS initialized during this script's execution (only run once)
    let osBackupDir;

    let pageInitDate = new Date();
    let pageInitStatus = localUserData.getStatus();

    $toolkitStatus.click(function() {
        console.log('Current App Status: ' + localUserData.getStatus() + "\n" +
            'Last Connection Check-in: ' + (localUserData.lastCheckDate ? localUserData.lastCheckDate : 'N/A') + "\n" +
            'Last Game Plan Login: ' + (localUserData.lastLoginDate ? localUserData.lastLoginDate : 'N/A') + "\n" +
            'Page Init Timestamp: ' + pageInitDate + "\n" +
            'Info Timestamp: ' + new Date() + "\n" +
            'Active Course ID: ' + localUserData.courseInSession.productID + "\n" +
            'Pulse User ID: ' + localUserData.userID);

        if (ctrlDown) {
            initToolkitModal();
        }
    });

    window.addEventListener("keydown", function(event) {
        if (event.keyCode === 17 && ctrlDown === false) {
            ctrlDown = true;
            $toolkitStatus.addClass('interactive');
        }
    }, false);

    window.addEventListener("keyup", function(event) {
        if (event.keyCode === 17 && ctrlDown === true) {
            ctrlDown = false;
            $toolkitStatus.removeClass('interactive');
        }
    }, false);

    // iff User is currently attending a week of camp AND pre-conditions are met for attempting a connection...
    if (debug || localUserData.shouldAttemptConnection()) {
        console.log('Running Toolkit Web Startup.');
        //webService = new WebService();
        initConnection();
        updateAppData();
        // TODO: Consider calling displayNotification() for a new logged in User stating automated backups are occuring.
    } else if (pageInitStatus !== localUserData.getStatus()) { // Status changed while checking pre-conditions...
        $toolkitStatus.removeClass().addClass(localUserData.getStatus()); // Update UI to reflect the change
    }

    /**
     * Initialize the Toolkit Modal with GP/Pulse data combined with App directory and backup data.
     */
    function initToolkitModal()
    {
        if (!osDirectories || !osBackupDir) {
            parseDirectories(localUserData.studentName, directories, function (err, dirs) {
                osDirectories = dirs;
                getBackupDirectory(function(err, dir) {
                    osBackupDir = dir;
                    displayToolkitModal();
                });
            });
        } else {
            displayToolkitModal();
        }

        // TODO: Pull Backup and Source Directories from the App for display too?
    }

    /**
     * Initialize the Toolkit Modal with GP/Pulse data combined with App directory and backup data.
     */
    function displayToolkitModal()
    {
        let osDirectoriesList = $('<ul></ul>');
        osDirectories.forEach(function(path) {
            osDirectoriesList.append('<li><a class="directory-link" data-path="' + path + '">' + path + '</a></li>');
        });

        $toolkitModal
            .addClass('show')
            .off() // kill all previously registered event handlers before initializing
            .on('click', function(e) { // Trigger on all click events for the Modal
                if ($(e.target).is($toolkitModal)) { // The click target is the parent and no children (i.e. background)
                    $toolkitModal.removeClass('show');
                } else { // A child element of the Modal
                    e.stopPropagation(); // Handle piecemeal per element
                }
            })
            .on('click', '#toolkit-cmd-backup', function (e) {
                e.preventDefault();
                if (!$(this).hasClass('disabled')) {
                    displayBackupPopover();
                }
            })
            .on('click', '#toolkit-cmd-restore', function (e) {
                e.preventDefault();
                if (!$(this).hasClass('disabled')) {
                    displayRestorePopover();
                }
            })
            .on('click', '#toolkit-cmd-usb', function (e) {
                e.preventDefault();
                if (!$(this).hasClass('disabled')) {
                    displayLoadUsbPopover();
                }
            })
            .on('click', '.directory-link', function(e) { // All Directory listing links under Backup/Source Directories
                e.preventDefault();
                let path = $(e.target).attr('data-path');
                connection.sendCommand('toolkit.directories.open', [path], function () {
                    console.log('Open Directory: ' + path);
                });
            })
            .find('#toolkit-user-backup')
                .empty()
                .append('<a class="directory-link" data-path="' + osBackupDir + '">' + osBackupDir + '</a>')
                .end()
            .find('#toolkit-user-sources')
                .empty()
                .append(osDirectoriesList);

        // TODO: Pull Backup and Source Directories from the App for display too
        // TODO: Split out into function - maybe initializeToolkitModal() ?
    }

    /**
     * Send Commands to Toolkit App Server based on GP Metadata (i.e. logged in User). Should only be refreshed on an
     * infrequent basis due to the number of Web Services called, and # of communication attempts.
     */
    function updateAppData()
    {
        // User is registered and logged in to Game Plan. Save relevant data to Toolkit App.
        // This is so automatic backups can still occur WITHOUT an internet connection when the App server runs.
        //let studentCode = $('#student_code').val(); // TODO: Remove dependency when Toolkit Web is integrated fully into GP Web.
        //let targetDate = $('#target_date').val(); // Debug Time Override (set during login) OR Current Server Time.
        //let displayName = userDataJson['displayName'];

        console.log('GP Metadata for Current User: ', localUserData.userID, localUserData.studentName);

        // Create Local Student Metadata file.
        let data = {
            //student_code: studentCode, // TODO: See above student code TODO comment.
            pulse_user_id: localUserData.userID,
            //target_date: targetDate.substring(0, 10), // Make sure we are only saving YYYY-MM-DD.
            directories: directories, // TODO: Ensure "%DESKTOP%/%STUDENT%" is ["%DESKTOP%/%STUDENT%"]
            student_name: localUserData.studentName,
            course_id: localUserData.courseInSession.productID,
            //setup: true // TODO: See above student code TODO comment.
        };

        writeUserData(data, function(err) {
            if (err) {
                console.error('Failed to write to data.json file. Try again later.', data, err);
            } else { // Metadata file successfully written. Refresh Toolkit App Auto Backups.
                console.log('Successfully written to data.json file. Attempting to refresh Automatic Backup Commands.');
                parseDirectories(data.student_name, data.directories, function (err, dirs) {
                    dirs.forEach(function(dir) { // Ensure all the directories exist. Okay to run in parallel.
                        console.log('toolkit.directories.ensureDir', dir);
                        connection.sendCommand('toolkit.directories.ensureDir', [dir]);
                    });

                    osDirectories = dirs;

                    // Refresh the auto backup interval for the User's data.
                    whenReady(function() {
                        let checkAutoData = {
                            directories: dirs,
                            pulse_user_id: data.pulse_user_id,
                            student_name: data.student_name,
                            //reset_interval: true // TODO: Fix Toolkit App getting stuck on resetting interval.
                        };

                        console.log('toolkit.backup.checkAuto', checkAutoData);
                        connection.sendCommand('toolkit.backup.checkAuto', [checkAutoData], function (/*results*/) {
                            // TODO: Toolkit App issue - no callback returned on success, only errors.
                            //console.log('toolkit.backup.checkAuto Completed:', results);
                        }, function (err) {
                            console.error('toolkit.backup.checkAuto Error:', err);
                        });
                    });
                });
            }
        });

        // Create Local Student Folder on Desktop. TODO: handle Courses with NoBackup set.
        whenReady(function() {
            connection.sendCommand('toolkit.directories.getDesktopDirectory', [], function (dir) {
                dir = dir.replace(/\\/g, "/");
                let studentDesktopPath = dir + '/' + data.student_name;
                connection.sendCommand('toolkit.directories.createDirectory', [studentDesktopPath], function () {

                    // TODO: Potentially run on first login of the week?
                    /*displayNotification(
                        'Welcome to Game Plan!',
                        "Here is your main project folder: " + studentDesktopPath,
                        'info',
                        '',
                        '',
                        true,
                        function (event) {
                            if (event) {
                                connection.sendCommand('toolkit.directories.open', [studentDesktopPath], function () {
                                    console.log('Open Directory: ' + studentDesktopPath);
                                });
                            }
                        }
                    );*/

                    connection.sendCommand('toolkit.directories.createDirectory', [studentDesktopPath], function () {

                    });
                    console.log('toolkit.directories.createDirectory successful @: ' + studentDesktopPath);
                }, function (err) {
                    console.error('toolkit.directories.createDirectory Error:', err);
                });
            }, function (err) {
                console.error('toolkit.directories.getDesktopDirectory Error:', err);
            });
        });
    }

    /**
     * Request to the Toolkit App Server to write to the data.json file with the given contents.
     * @param data
     * @param {function (err:string?)} callback
     */
    function writeUserData(data, callback)
    {
        console.log('Attempting to save user data.');
        whenReady(function () {
            connection.sendCommand('toolkit.config.write', [data || {}], function () {
                console.log('toolkit.config.write successful..');
                callback();
            }, function (err) {
                console.log('toolkit.config.write error.', err);
                callback(err);
            });
        });
    }

    /**
     *
     */
    function getBackupDirectory(callback)
    {
        whenReady(function () {
            connection.sendCommand('toolkit.backup.getBackupDirectory', [], function (dir) {
                callback(null, dir);
            }, function (err) {
                callback(err);
            });
        });
    }

    /**
     * Parses the special characters in the directory.
     * @param studentName Student's name used for Desktop folder.
     * @param encodedDirs Encoded directory paths. (i.e. %DESKTOP%/%STUDENT%)
     * @param {function (err:string|null?, dirs:string[]?)} callback
     * @return {string}
     */
    function parseDirectories(studentName, encodedDirs, callback)
    {
        whenReady(function () {
            /**
             * @param osDirs Operating System specific directory paths. (i.e. Windows == C:/Users/Student/Desktop)
             */
            connection.sendCommand('toolkit.directories.getDirectories', [], function (osDirs) {
                for (let i = 0, len = encodedDirs.length; i < len; i++) {
                    encodedDirs[i] = parseDirectory(encodedDirs[i]);
                }

                callback(null, encodedDirs);

                /**
                 *
                 * @param dir
                 * @returns {string}
                 */
                function parseDirectory(dir)
                {
                    dir = (dir || '')
                        .replace(/%DESKTOP%/g, osDirs.desktop)
                        .replace(/%APPDATA%/g, osDirs.appdata)
                        .replace(/%APPDATA_MINECRAFT%/g, osDirs.appdata_minecraft)
                        .replace(/%DOCUMENTS%/g, osDirs.documents)
                        .replace(/%STUDENT%/g, studentName)
                        .replace(/%STUDENT_SLUG%/g, studentName.replace(/\s|[^a-z]/gi, '').toLowerCase());

                    if (osDirs.platform === 'win32') { // If Windows, default to all backwards slashes. This is to avoid mixed use cases like "C:\Users\Student/iD Tech/backups"
                        return dir.replace(/\//g, "\\");
                    } else { // else, default to the Unix/Linux standard for forwards slashes. (Likely no change if entered correctly in Pulse)
                        return dir.replace(/\\/g, "/");
                    }
                }

            }, function (err) {
                callback(err);
            });
        });
    }

    /**
     * Displays a notification to the client.
     * @param {string} title
     * @param {string} message
     * @param {string} [type='info'] The type of notification to show. Available: info, success, error, warn
     * @param {string} [subtitle='']
     * @param {string} [open=''] A URL to open upon clicking the notification.
     * @param {boolean} [wait=false] If true, callback is not called until the notification is closed.
     * @param {function (event:string?)=} callback
     */
    function displayNotification(title, message, type, subtitle, open, wait, callback)
    {
        whenReady(function () {
            connection.sendCommand('toolkit.notifier.notify', [{type:type, title:title, message:message, subtitle:subtitle, open:open, wait:wait}], callback);
        });
    }

    /**
     * Initialize the Toolkit App Connection.
     */
    function initConnection()
    {
        console.log('Initializing Toolkit App connection on port [' + port + '] with protocol [' + protocol + ']...');
        $toolkitStatus.removeClass().addClass(LocalUserData.STATUS_CONNECTING);
        localUserData.setStatus(LocalUserData.STATUS_CONNECTING);

        // Loop to keep our connection to Toolkit open.
        // Only kicks in if we've previously had a successful connection or they manually hit a button to trigger one.
        connectionIntervalId = setInterval(function () {
            if (connectionAttempted && (!connection.ready && !connection.connecting)) {
                console.log('Attempting to connect.');
                connection.connect({port: port, protocol: protocol});
            }
        }, connectionInterval);

        connection = new Connection();

        // Try once on startup.
        connection.connect({port: port, protocol: protocol});

        // Setup our default listeners.
        connection.addListener('open', function (e) {
            console.log('Toolkit App Connection successfully opened!');
            $toolkitStatus.removeClass().addClass(LocalUserData.STATUS_CONNECTED);
            localUserData.setStatus(LocalUserData.STATUS_CONNECTED);
            failedAttempts = 0;
            connectionAttempted = true;
        });

        connection.addListener('message', function (e) {
            let connectionData = JSON.parse(e.data);
            console.log('Toolkit App Connection message:', connectionData, e);
            if (connectionData.type === 'error') {
                console.error('Connection message (non-fatal):', e);
                document.dispatchEvent(new Event('connection_error'));
            }
        });

        connection.addListener('close', function (e) {
            console.log('Toolkit App Connection closed!');
            $toolkitStatus.removeClass().addClass(LocalUserData.STATUS_ERROR);
            localUserData.setStatus(LocalUserData.STATUS_ERROR);
        });

        connection.addListener('error', function (e) {
            console.log('Toolkit App Connection error!');
            failedAttempts++;

            if (failedAttempts >= failedAttemptLimit) {
                clearInterval(connectionIntervalId);
                console.log('Failed limit reached. Aborting...');
                console.log("There was a problem connecting to the Toolkit App. Please try refreshing the page.");
            }
        });
    }

    /**
     * Calls the callback function only when a valid connection is available.
     * This is a safety function to ensure we're connected before trying things.
     * It's good to wrap the contents of functions that need to communicate with the
     * client in this.
     * @param {function} callback
     */
    function whenReady(callback)
    {
        if (!connection || !connection.ready) {
            setTimeout(function () {
                whenReady(callback);
            }, connectionInterval);
        } else {
            callback();
        }
    }

    /**
     * Register click events here to keep things clean.
     * Remember to use the body class(es) as selectors and use $(document).on()
     * instead of $(selector).click, or they won't get hooked properly.
     */
    function registerClickEvents()
    {
        /*$toolkitModal
            .off() // kill all previously registered event handlers before initializing
            .on('click', 'a, button', function (e) { // global button events (before trickling down to specific events)
                if ($(this).hasClass('disabled')) {
                    e.preventDefault();
                    e.stopPropagation();
                }
            }).on('click', '.buttons .backup', function (e) {
                e.preventDefault();
                if (!$(this).hasClass('disabled')) {
                    displayBackupPopover();
                }
            }).on('click', '.buttons .restore', function (e) {
                e.preventDefault();
                if (!$(this).hasClass('disabled')) {
                    userData.get('backups', function (err, data) {
                        if (err) {
                            displayMessage(err, 'error');
                        } else {
                            displayRestorePopover(data);
                        }
                    });
                }
            }).on('click', '.sources span', function(e) {
                e.preventDefault();
                let path = $(e.target).text();
                connection.sendCommand('toolkit.directories.open', [path], function () {
                    console.log('Open Directory: ' + path);
                });
            });*//*.on('click', 'body.main-menu .button.load-usb', function(e) {
                e.preventDefault();
                if (!$(this).hasClass('disabled')) {
                    displayLoadUsbPopover();
                }
            }).on('click', 'body.main-menu .student-info .icon', function(e) {
                launchTechbot();
            });*/

            /*if (config.debug) {
                $(document).on('click', 'body.main-menu .debug.notify-info', function (e) {
                    displayNotification('Test Info', 'Test Message', 'info', 'Test Subtitle');
                }).on('click', 'body.main-menu .debug.notify-warn', function (e) {
                    displayNotification('Test Warn', 'Danger! Danger!', 'warn', 'Test Subtitle');
                }).on('click', 'body.main-menu .debug.notify-error', function (e) {
                    displayNotification('Test Error', 'Test Message', 'error', 'Test Subtitle');
                }).on('click', 'body.main-menu .debug.notify-success', function (e) {
                    displayNotification('Test Success', 'Test Message', 'success', 'Test Subtitle');
                }).on('click', 'body.main-menu .debug.open-backup-directory', function (e) {
                    connection.sendCommand('toolkit.backup.getBackupDirectory', [], function (path) {
                        connection.sendCommand('toolkit.directories.open', [path], function () {});
                    });
                }).on('click', 'body.main-menu .debug.clean-backups', function (e) {
                    connection.sendCommand('toolkit.backup.cleanup', [], function () {
                        console.log('Backups cleanup completed.');
                    }, function (err) {
                        console.error(err);
                    });
                }).on('click', 'body.main-menu .debug.check-same-backups', function (e) {
                    userData.get(['pulse_user_id', 'directories'], function (err, data) {
                        if (err) {
                            displayMessage(err, 'error');
                        } else {
                            parseDirectories(data.directories, function(err, directories) {
                                if (err) {
                                    displayMessage(err, 'error');
                                } else {
                                    whenReady(function() {
                                        connection.sendCommand('toolkit.backup.checkSame', [{directories: directories, pulse_user_id: data.pulse_user_id}], function (results) {
                                            console.log('toolkit.backup.checkSame Results:', results);
                                        }, function (err) {
                                            console.error('toolkit.backup.checkSame Error:', err);
                                        });
                                    });
                                }
                            });
                        }
                    });
                }).on('click', 'body.main-menu .debug.check-auto-backup', function (e) {
                    userData.get(['pulse_user_id', 'directories'], function (err, data) {
                        if (err) {
                            displayMessage(err, 'error');
                        } else {
                            parseDirectories(data.directories, function(err, directories) {
                                if (err) {
                                    displayMessage(err, 'error');
                                } else {
                                    whenReady(function() {
                                        connection.sendCommand('toolkit.backup.checkAuto', [{directories: directories, pulse_user_id: data.pulse_user_id}], function (results) {
                                            console.log('toolkit.backup.checkAuto Completed:', results);
                                        }, function (err) {
                                            console.error('toolkit.backup.checkAuto Error:', err);
                                        });
                                    });
                                }
                            });
                        }
                    });
                }).on('click', 'body.main-menu .debug.validate-directories', function (e) {
                    userData.get(['pulse_user_id', 'directories'], function (err, data) {
                        if (err) {
                            displayMessage(err, 'error');
                        } else {
                            validateDirectories();
                        }
                    });
                }).on('click', 'body.main-menu .debug.app-update', function (e) {
                    checkVersion(true);
                });
            }*/
    }

    /**
     *
     * @param callback
     */
    function getExternalIp(callback) {
        $.getJSON("https://api.ipify.org?format=jsonp&callback=?", function(json) {
            callback(json.ip);
        });
    }

    /**
     *
     * @param ip
     * @param callback
     */
    function getGeoLocationData(ip, callback) {
        $.getJSON("http://ip-api.com/json/" + ip, function(json) {
            callback(json);
        });
    }

    /**
     *
     * @param {string} type 'manual', or 'auto'.
     * @param callback
     */
    function createBackup(type, callback)
    {
        type = type || 'manual';
        whenReady(function() {
            connection.sendCommand('toolkit.backup.backup', [{directories: osDirectories, pulse_user_id: localUserData.userID, type: type}], function (data) {
                callback(null, data);
            }, function(err) {
                callback(err);
            });
        });
    }

    /**
     * Attempt to load USB for the given destination directory.
     *
     * @param dest
     * @param free
     * @param callback
     */
    function loadUsb(dest, free, callback)
    {
        let srcs = [];
        let dests = [];

        // Retrieve all sources first; then copy over each asynchronously. Callback called once all have been processed.
        for (let i = 0; i < osDirectories.length; i++) {
            srcs.push(osDirectories[i]);
            dests.push(osDirectories[i].split(/\\|\//g).pop() + '/');
        }

        console.log('USB Source Directories retrieved.');
        console.log('srcs: ', srcs); // Full directory paths i.e. ("C:/Users/Student/Desktop/Student Name", etc.)
        console.log('dests: ', dests); // Capstone directories only. i.e. ("Student Name/", "saves/")

        let fileContent = getReadmeContent();
        let filePath = srcs[0] + '/README-' + localUserData.courseInSession.productShortName + '.html'; // /Users/Student/Desktop/Student Name/README-CSGO3.html

        console.log('fileContent: ', fileContent);
        console.log('filePath: ', filePath);

        whenReady(function() {
            connection.sendCommand('toolkit.directories.createFile', [{
                fileContent: fileContent,
                filePath: filePath
            }], function () {
                console.log('Finished request to create text file at: ', filePath);
                whenReady(function() {
                    connection.sendCommand('toolkit.directories.sizeDirs', [srcs], function (size) { // sanity check; before copying, check if there is enough space
                        console.log(size, free);
                        if (size > free) {
                            console.error(err);
                            callback(bytesToSize(size) + " required. " + bytesToSize(free) + " free.");
                        } else {
                            whenReady(function() {
                                connection.sendCommand('toolkit.directories.copyAllIndividually', [{
                                    srcs: srcs,
                                    dests: dests,
                                    absPathDest: dest
                                }], function () {
                                    console.log('copyAllIndividually success');
                                    callback();
                                }, function (err) {
                                    callback(err); // Tally individual directory errors since we're past the point of no return.
                                });
                            });
                        }
                    }, function (err) {
                        console.error("Failed to tally total project size.", err);
                        callback(err);
                    });
                });
            }, function (err) {
                console.error("Failed to tally total project size.", err);
                callback(err);
            });
        });
    }

    /**
     *
     * @param callback
     */
    function getUserBackups(callback)
    {
        console.log('Retriving list of backups for User: ' + localUserData.userID);

        whenReady(function () {
            connection.sendCommand('toolkit.backup.list', [{pulse_user_id: localUserData.userID}], function (data) {
                console.log('Successfully retrieved backup data: ', data.backups);

                let backups = data.backups; // Array[#] {filepath: "...", created: "..."}
                /*let latest = null; // Determine the last backup.

                /!** @type {file_path:string, created:string} backup *!/
                backups.forEach(function (backup) {
                    console.log('Processing ' + backup);
                    backup.created = new Date(backup.created);

                    if (!latest || (backup.created && latest.created && backup.created.getTime() > latest.created.getTime())) {
                        latest = backup;
                    }
                    console.log('Latest: ', latest);
                });*/

                /*console.log('refreshBackups ==1== latest', latest);
                self.sources.backups.data.last_backup = latest;
                self.sources.backups.created = Date.now();
                console.log('refreshBackups ==2== callback()');*/
                callback(backups);
            }, function (err) {
                callback(err);
            });
        });
    }

    /**
     *
     * @returns {string} The USB README contents.
     */
    function getReadmeContent()
    {
        let hrefReadme = "https://www.idtech.com/readme/" +
            new Date(localUserData.courseInSession.sessionStartDate).getFullYear() + "-" +
            localUserData.courseInSession.productShortName.toLowerCase();
        let hrefMore = "https://www.idtech.com/more";

        return "<!-- " + "\n" +
            "README file generated by iD Toolkit on " + new Date() + "\n" +
            "Pulse User ID: " + localUserData.userID + "\n" +
            "Student Name: " + localUserData.studentName + "\n" +
            "Course ID: " + localUserData.courseInSession.productID + "\n" +
            "Session ID: " + localUserData.courseInSession.sessionID + "\n" +
            "-->" + "\n" +
            "<!DOCTYPE html>" + "\n" +
            "<html>" + "\n" +
            "<body>" + "\n" +
            "<strong>" + localUserData.courseInSession.locationName + "</strong>" + "\n" +
            "<p>Congrats on completing:</p>" + "\n" +
            "<strong>" + localUserData.courseInSession.productName + "</strong>" + "\n" +
            "<p>For specific information on using your course files at home, " +
            "please head to <a href='" + hrefReadme + "'>" + hrefReadme + "</a><br/></p>" + "\n" +
            "<p>Wondering what to do next? We've got you covered!<br/>" +
            "Head to <a href='" + hrefMore + "'>" + hrefMore + "</a></p>" + "\n" +
            "</body>" + "\n" +
            "</html>" + "\n" +
            "<!--" + "\n" +
            "Made with:" + "\n" +
            "   ( (" + "\n" +
            "    ) )" + "\n" +
            "  ........" + "\n" +
            "  |      |]" + "\n" +
            "  \\      /" + "\n" +
            "   `----'" + "\n" +
            "-->";
    }

    /**
     * Displays a message the top of the page.
     * @param {string} message The message to display.
     * @param {string} [html_class="notice"] A class to add to the message, in addition to "message", for styling. Available: notice, success, error
     */
    /*function displayMessage (message, html_class)
    {
        console.log('Displaying Message', message, html_class);
        if (Array.isArray(message)) {
            message = message.join('<br>');
        }
        if ($body.hasClass('setup') || $body.hasClass('evals')) {
            $('.error').empty().append(message);
        } else {
            $('<div class="message">')
                .addClass(html_class || 'notice')
                .insertAfter('#connection-status')
                .append(message);
        }
    }*/

    /**
     * User-triggered/Manual Backup Toolkit Popover UI component and logic.
     */
    function displayBackupPopover()
    {
        let $popover = $('#toolkit-popover');
        if ($popover.length) {
            return;
        }

        var $buttons = $('<ul class="row"></ul>')
            .append($('<li class="col-6"></li>')
                .append($('<button class="btn btn-outline-primary btn-lg btn-block">OK</button>')
                    .click(function () {
                        if (!$(this).hasClass('disabled')) {
                            console.log('Creating Manual Backup');

                            $('#toolkit-popover') // Trigger some UI/UX notifications for User.
                                .find('button')
                                    .addClass('disabled').attr('disabled', true) // disable both buttons
                                    .first().empty().append('<span class="spinner-border"></span>') // loading indicator
                                    .end().end() // back to parent element
                                .find('.info')
                                    .text('Backing up project, please wait...'); // text status for user

                            setTimeout(function() { // before backup; small delay to allow feedback on smaller projects
                                createBackup('manual', function (err, data) {
                                    if (!err) {
                                        console.log('Manual Backup Created', data);
                                        displayNotification('Backup Created', new Date().toString(), 'success');
                                        finalizePopover('Backup created!<br/>' + data.file_path + '<br/>' + new Date(data.created));
                                    } else {
                                        console.log('Manual Backup Error', err);
                                        displayNotification('Backup Failed', new Date().toString(), 'fail');
                                        finalizePopover('Backup failed.');
                                    }
                                });
                            }, uiFeedbackDelay);
                        }
                    })))
            .append($('<li class="col-6"></li>')
                .append($('<button class="btn btn-outline-secondary btn-lg btn-block">Cancel</button>')
                    .click(function () {
                        if (!$(this).hasClass('disabled')) {
                            $('#toolkit-popover').remove();
                        }
                    })));

        displayPopover('Backup Project?', $buttons);
    }

    /**
     * User-triggered/Manual Restore Toolkit Popover UI component and logic.
     */
    function displayRestorePopover()
    {
        let $popover = $('#toolkit-popover');
        if ($popover.length) {
            return;
        }

        let $buttons = $('<ul class="row"></ul>')
            .append($('<li class="col-6"></li>')
                .append($('<button class="btn btn-outline-primary btn-lg btn-block disabled" disabled>OK</button>')
                    .click(function () {
                        if (!$(this).hasClass('disabled')) {
                            let $popover = $('#toolkit-popover');
                            let backup = $popover.find('select').val();

                            console.log('Restoring backup', backup);
                            console.log('Creating backup before restore.');

                            $popover
                                .find('select, button').addClass('disabled').attr('disabled', true).end() // disable all inputs
                                .find('button').first().empty().append('<span class="spinner-border"></span>').end().end() // loading indicator
                                .find('.info').empty().append('Restoring project, please wait...<br>This process will take some time for larger projects.');

                            setTimeout(function() { // small delay for feedback on small projects
                                createBackup('manual', function (err, data) {
                                    if (err) {
                                        console.log('Manual Backup Error (before Restore).', err);
                                        displayNotification('Backup Failed', new Date(), 'fail');
                                        finalizePopover('Backup before restore failed.');
                                    } else {
                                        console.log('Manual Backup Created (before Restore).', data);
                                        displayNotification('Backup Created', new Date(), 'success');
                                        connection.sendCommand('toolkit.backup.restore', [backup], function () {
                                            console.log('Restore Project completed.');
                                            displayNotification('Backup Restored', new Date(), 'success');
                                            finalizePopover('Restored project!<br/>Please confirm by checking for the expected files within your Student\'s Source Directories.');
                                        }, function (err) {
                                            console.log('Restore Project failed.', err);
                                            finalizePopover('Restore project failed.');
                                        });
                                    }
                                });
                            }, uiFeedbackDelay);
                        }
                    })))
            .append($('<li class="col-6"></li>')
                .append($('<button class="btn btn-outline-primary btn-lg btn-block">Cancel</button>')
                    .click(function () {
                        if (!$(this).hasClass('disabled')) {
                            $('#toolkit-popover').remove();
                        }
                    })));

        displayPopover(
            'First, close all applications, then select a backup to restore:',
            $buttons,
            '<div class="form-control"><span class="spinner-border"></span></div>'
        );

        // Trigger async operation to retrieve backups after UI initialized
        setTimeout(function() { // small delay for feedback on small projects
            getUserBackups(function(backups) {
                let $select = $('<select class="form-control"></select>');
                for (let i = backups.length - 1; i >= 0; i--) { // most recent first
                    if (backups.hasOwnProperty(i)) {
                        $select.append('<option value="' + backups[i].file_path + '">' + new Date(backups[i].created) + '</option>');
                    }
                }

                let $popover = $('#toolkit-popover');
                if ($popover.length) {
                    $popover
                        .find('div.form-control').replaceWith($select).end()
                        .find('button').removeClass('disabled').removeAttr('disabled');
                }
            });
        }, uiFeedbackDelayShort);
    }

    /**
     * User-triggered/Manual Load USB Toolkit Popover UI component and logic.
     */
    function displayLoadUsbPopover()
    {
        let $popover = $('#toolkit-popover');
        if ($popover.length) {
            return;
        }

        let $buttons = $('<ul class="row"></ul>')
            .append($('<li class="col-6"></li>')
                .append($('<button class="btn btn-outline-primary btn-lg btn-block disabled" disabled>OK</button>')
                    .click(function () {
                        if (!$(this).hasClass('disabled')) {
                            let $popover = $('#toolkit-popover');
                            let dest = $popover.find('select').val();
                            let free = parseInt($popover.find('select').find('option:selected').attr('data-free'));

                            console.log('Loading files onto selected drive: ', dest, free);

                            $popover
                                .find('select, button').addClass('disabled').attr('disabled', true).end() // disable all inputs
                                .find('button').first().empty().append('<span class="spinner-border"></span>').end().end() // loading indicator
                                .find('.info').empty().append('Loading files...<br>Please do not remove device.');

                            setTimeout(function() { // small delay for feedback on small projects
                                document.addEventListener('connection_error', loadUsbConnectionErrorListener);
                                loadUsb(dest, free, function (err) {
                                    if (!err) {
                                        console.log('Load USB files loaded successfully.');
                                        finalizePopover('Loaded files successfully!<br/>Please confirm by checking for the expected files within your Student\'s USB drive.');
                                    } else {
                                        console.error(err);
                                        finalizePopover('Error: Could not load files. ' + err);
                                    }
                                    document.removeEventListener('connection_error', loadUsbConnectionErrorListener);
                                });
                            }, uiFeedbackDelay);
                        }
                    })))
            .append($('<li class="col-6"></li>')
                .append($('<button class="btn btn-outline-primary btn-lg btn-block">Cancel</button>')
                    .click(function () {
                        if (!$(this).hasClass('disabled')) {
                            $('#toolkit-popover').remove();
                        }
                    })));

        displayPopover(
            'Please select a drive:',
            $buttons,
            '<div class="form-control"><span class="spinner-border"></span></div>'
        );

        setTimeout(function() { // small delay for feedback on small projects
            whenReady(function() {
                connection.sendCommand('toolkit.directories.getDriveListFree', [], function (drives) {
                    console.log(drives);
                    let $select = $('<select class="form-control"></select>');
                    let exclude = ['C:', '/Volumes/Macintosh HD', '/Volumes/OSX'];

                    for (let i = 0; i < drives.length; i++) {
                        if (drives[i] && drives[i]['mountpoint'] && exclude.indexOf(drives[i]['mountpoint']) === -1) { // Exclude internal camp drives.
                            $select.append('<option value="' + drives[i]['mountpoint'] + '" data-free="' + drives[i]['diskspace']['free'] + '">'
                                + drives[i]['mountpoint'] + ' (' + bytesToSize(drives[i]['diskspace']['free']) + ' free of ' + bytesToSize(drives[i]['diskspace']['total']) + ')' +
                                '</option>');
                        }
                    }


                    let $popover = $('#toolkit-popover');
                    if ($popover.length) {
                        if ($select.children('option').length > 0) {
                            $popover
                                .find('div.form-control').replaceWith($select).end()
                                .find('button').removeClass('disabled').removeAttr('disabled');
                        } else {
                            finalizePopover('No free drives found.<br>Please insert a USB drive and try again.');
                        }
                    }
                }, function (err) {
                    console.error(err);
                    displayNotification('Error!', 'Could not load drives.', 'error');
                });
            });
        }, uiFeedbackDelayShort);
    }

    /**
     *
     * @param e
     */
    function loadUsbConnectionErrorListener(e)
    {
        console.error(e);
        finalizePopover('Could not load files.');
        document.removeEventListener('connection_error', loadUsbConnectionErrorListener);
    }

    /**
     * Display a Toolkit Popover in conjunction with the Toolkit Modal UI. This takes display priority over the Modal.
     * Usually contains interactable elements that can/will be modified on the fly.
     *
     * @param message
     * @param buttons
     * @param content
     */
    function displayPopover (message, buttons, content)
    {
        message = message || '';
        buttons = buttons || $('<button class="centered fill">OK</button>').on('click', function(){ $('#toolkit-popover').remove(); });
        content = content || '';

        let $popover =
            $('<div id="toolkit-popover" class="popover"></div>')
                .append($('<div class="wrapper"></div>')
                    .append($('<p class="info"></p>')
                        .append(message))
                    .append(content)
                    .append(buttons));

        $navbar.append($popover);
    }

    /**
     * Used when a Toolkit Popover is already displayed, but the task it was responsible for is finished.
     *
     * @param {string} message
     * @param {string} [button_text="Back to Menu"]
     */
    function finalizePopover (message, button_text)
    {
        let $popover = $('#toolkit-popover');
        if ($popover.length) {
            $popover
                .find('button').remove().end()
                .find('.info').empty().append(message).end()
                .find('ul').append($('<li class="col-md-12"></li>')
                .append($('<button class="btn btn-outline-primary btn-lg btn-block"></button>')
                    .text(button_text || 'Back to Menu')
                    .click(function(){
                        $popover.remove();
                    })));
        }
    }

    /**
     * Loads the server and client manifests, then checks if they have the same version.
     * If they don't, triggers updateClient()
     */
    /*function checkVersion (updateOverride)
    {
        updateOverride = updateOverride || false;
        whenReady(function () {
            console.log('Checking version.');
            var serverManifest,
                clientManifest;

            function doCheck() {
                console.log('Server version: ' + serverManifest.version + ' | Client version: ' + clientManifest.version);
                if (serverManifest.version !== clientManifest.version || updateOverride) {
                    console.log('Client needs an update, beginning...');
                    $updateStatus.attr('class', '').addClass('updating');
                    $updateScreen.attr('class', 'show').addClass('animated fadeIn');
                    updateClient(serverManifest, clientManifest);
                } else {
                    console.log('Client up-to-date');
                    $updateStatus.attr('class', '').addClass('up-to-date');
                }
            }

            $.ajax('/manifest', {
                dataType: 'json'
            }).done(function (manifest) {
                serverManifest = manifest;

                if (clientManifest) {
                    doCheck();
                }
            }).fail(function (err) {
                console.log('Error reading manifest from server', err);
                $updateStatus.attr('class', '').addClass('error');
                $updateScreen.attr('class', '');
            });

            connection.sendCommand('toolkit.update.readManifest', [], function (manifest) {
                clientManifest = manifest;

                if (serverManifest) {
                    doCheck();
                }
            }, function () {
                console.log('Error reading manifest from client', arguments);
                $updateStatus.attr('class', '').addClass('error');
                $updateScreen.attr('class', '');
            });
        });
    }*/

    /**
     *
     * @param serverManifest
     * @param clientManifest
     */
    /*function updateClient (serverManifest, clientManifest)
    {
        console.log('Updating Client');

        var files = diffManifests(serverManifest, clientManifest);
        files.push({source: absolutePath('manifest'), dest: 'version.json'}); // this one has to be out of date

        console.log(files.length, "files to update.");

        whenReady(function () {
            console.log('Sending files', files);
            connection.sendCommand('toolkit.update.receiveFiles', [files], function () {
                console.log('Update complete.');
                setTimeout(function() { // for debug, wait at least 3 seconds before removing update screen
                    $updateStatus.attr('class', '').addClass('up-to-date');
                    $updateScreen
                        .find('h1').html('Files updated!<br>Please restart your computer for the changes to take effect.').end()
                        .find('span').removeClass('loading').end();
                }, 3000);
            }, function () {

            });
        });
    }*/

    /**
     * Diffs the manifests, and returns an array of objects which have a source and dest for each file.
     * @param {{}} serverManifest The server manifest (JSON object)
     * @param {{}} clientManifest The client manifest (JSON object)
     */
    /*function diffManifests(serverManifest, clientManifest)
    {
        var files = [];

        for (var filePath in serverManifest.files) {
            if (serverManifest.files.hasOwnProperty(filePath)) {
                if (!clientManifest.files[filePath] || clientManifest.files[filePath] != serverManifest.files[filePath]) {
                    files.push({source: absolutePath('update/' + filePath), dest: filePath});
                }
            }
        }

        return files;
    }*/

    /**
     * Makes a relative server path absolute for the current domain.
     * @param {string} path
     */
    function absolutePath (path) {
        return location.protocol + "//" + location.host + "/" + path.replace(/^\//, '');
    }

    /**
     * For debug only.
     *
     * @param dirs
     * @param callback
     */
    function validateDirectories (dirs, callback)
    {
        if (!dirs) {
            dirs = [
                '%DESKTOP%/%STUDENT%',
                '%DESKTOP%/%STUDENT%|%APPDATA_MINECRAFT%/saves|%APPDATA_MINECRAFT%/mods|%APPDATA_MINECRAFT%/resourcepacks',
                '%DESKTOP%/%STUDENT%|C:/Users/Public/Games/Runic Games/Torchlight 2/mods',
                '%DESKTOP%/%STUDENT%|C:\\Program Files (x86)\\Steam\\steamapps\\common\\dota 2 beta\\content\\dota_addons',
                '%DESKTOP%/%STUDENT%|%APPDATA%/Stencyl/stencylworks/games',
                '%DESKTOP%/%STUDENT%|%DOCUMENTS%/LEGO Creations/WeDo/Projects',
                '%DESKTOP%/%STUDENT%|%APPDATA_MINECRAFT%/saves|%APPDATA_MINECRAFT%/resourcepacks'
            ];
        }

        console.log(dirs);
        parseDirectories('FakenameMagoo', dirs.slice(0), function(err, parsedDirs) {
            console.log(parsedDirs);
        });
    }

    /**
     * Returns a string for the given Date, in the format: YYYY-MM-DD hh:ii AM/PM
     */
    /*function getFormattedDate(date)
    {
        date = date || new Date();
        let month = date.getMonth() + 1;
        let day = date.getDate();
        let hour = date.getHours();
        let min = date.getMinutes();
        let ampm = hour >= 12 ? 'PM' : 'AM';

        // AM/PM split for hours.
        hour = hour % 12;
        hour = hour ? hour : 12;

        // Padded zeroes.
        month = (month < 10 ? "0" : "") + month;
        day = (day < 10 ? "0" : "") + day;
        hour = (hour < 10 ? "0" : "") + hour;
        min = (min < 10 ? "0" : "") + min;

        return date.getFullYear() + "-" + month + "-" + day + " " +  hour + ":" + min + ' ' + ampm;
    }*/

    /**
     * Convert Bytes to a human readable string. Conversion uses binary representation, not decimal.
     *
     * @param bytes
     * @returns {string}
     */
    function bytesToSize(bytes)
    {
        let sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB'];
        if (bytes === 0) {
            return '0 Byte';
        }
        let i = Math.floor(Math.log(bytes) / Math.log(1024));
        return (bytes / Math.pow(1024, i)).toFixed(1) + ' ' + sizes[i];
    }

    /**
     *
     */
    /*function initPatternCheck()
    {
        $body.on('keydown', function(e) {
            checkPattern(e.which);
        });
    }*/

    /**
     *
     */
    /*function checkPattern(keyCode)
    {
        for (var i in patternsInput) {
            patternsInput[i] += keyCode;
            if (patternsInput[i].length > patternsFinal[i].length) {
                patternsInput[i] = patternsInput[i].substr((patternsInput[i].length - patternsFinal[i].length));
            }
        }

        if (patternsInput[0] == patternsFinal[0]) {
            patternsInput[0] = '';
            console.log('The Konami code! From before they became awful!');
            launchTechbot();
        } else if (patternsInput[1] == patternsFinal[1]) {
            patternsInput[1] = '';
            // Credit to fangamer.net's Undertale easter egg for this one.
            // http://www.fangamer.com/products/undertale-determination-combo
            console.log('Why can\'t skeletons play music? Because they have no organs.');
            var randomizeMe = '* Hey! You\'re looking for the easter egg right?';
            var randomized = '';
            for (var j = 0; j < randomizeMe.length; j++) {
                if (randomizeMe[j] != ' ') {
                    randomized += '<span class="randomize">' + randomizeMe[j] + '</span>';
                } else {
                    randomized += ' ';
                }
            }

            $body.addClass('ut-enabled').after($('<div class="container flowey"></div>')
                .append($('<div class="row"></div>')
                    .append($('<div class="col-sm-8 col-sm-offset-2"></div>')
                        .append('<div class="ut-random textbox">' + randomized + '</div>')))
                .append($('<div id="flowey-wink"><img src="/assets/images/flowey.png"></div>'))
                .append($('<div id="flowey-grin" class="ut-hide-offscreen"><img src="/assets/images/flowey_grin.png"></div>'))
                .append($('<div class="row"></div>')
                    .append($('<div class="col-sm-8 col-sm-offset-2"></div>')
                        .append($('<div class="ut-random textbox" style="padding-bottom:15px;margin-bottom:30px;">* This doesn\'t feel right.</div>')
                            .append($('<div class="row"></div>')
                                .append($('<div class="col-xs-3 col-xs-offset-1 text-center"></div>').append('<a href="#" class="ut-option-hover">YES</a>').on('click', function(e) {
                                    e.preventDefault();
                                    $('#flowey-wink').hide();
                                    $('#flowey-grin').removeClass('ut-hide-offscreen').addClass('animated fadeOut').one('webkitAnimationEnd oAnimationend oAnimationEnd msAnimationEnd animationend', function(e) {
                                        $body.removeClass('ut-enabled');
                                        $('.container.flowey').remove();
                                        enableShyButtons(); // do evil thing
                                    });
                                }))
                                .append($('<div class="col-xs-3 col-xs-offset-1 text-center"></div>')
                                    .append($('<a href="http://google.com?q=cartoons for babies or whatever" class="ut-option-hover" target="_blank">NO</a>').on('click', function() {
                                        $body.removeClass('ut-enabled');
                                        $('.container.flowey').remove();
                                    }))))))));

            randomize();
        }
    }*/

    /**
     *
     * @param min
     * @param max
     * @returns {*}
     */
    /*function random(min, max)
    {
        if (max == null) {
            max = min;
            min = 0;
        }

        return min + Math.floor(Math.random() * (max - min + 1));
    }*/

    /**
     *
     */
    /*function randomize()
    {
        $('.randomize').each(function() {
            var $this = $(this);
            var num = random(-1600, 106);
            if (num <= 90) {
                $this.css('top', '0');
                $this.css('left', '0');
            }
            else if (num <= 92) {
                $this.css('top', '1px');
            }
            else if (num <= 94) {
                $this.css('top', '-1px');
            }
            else if (num <= 96) {
                $this.css('left', '1px');
            }
            else if (num <= 98) {
                $this.css('left', '-1px');
            }
            else if (num <= 100) {
                $this.css('top', '1px');
                $this.css('left', '1px');
            }
            else if (num <= 102) {
                $this.css('top', '-1px');
                $this.css('left', '1px');
            }
            else if (num <= 104) {
                $this.css('top', '1px');
                $this.css('left', '-1px');
            }
            else if (num <= 106) {
                $this.css('top', '-1px');
                $this.css('left', '-1px');
            }
        });

        randomTimeout = setTimeout(randomize, 1000 / 10);
    }*/

    /**
     *
     */
    /*function enableShyButtons()
    {
        $('button, .button')
            .css({transition: 'all 0.25s ease-in-out', left: '0px', top: '0px', zIndex: 9000})
            .on('mouseover', function() {
                let $this = $(this);
                let width = $this.outerWidth();
                let height = $this.outerHeight();
                let left = random(-50, 50);
                let top = random(-50, 50);
                let zIndex = (parseInt($this.css('zIndex')) + 1);

                left = (left >= 0 ? width + left : -width + left);
                top = (top >= 0 ? height + top : -height + top);

                $this.css({
                    left: left + "px",
                    top: top + "px",
                    zIndex: zIndex
                });
            });
    }*/

    /**
     *
     */
    /*function launchTechbot()
    {
        let direction;
        let $techbot = $('#techbot');
        if ($techbot.length > 0) {
            $techbot.remove();
        }

        $techbot = $('<img id="techbot" src="assets/images/tech-bot-animation.gif">').hide();
        $('body').after($techbot);

        direction = random(0, 2);
        if (direction === 0) {
            $techbot.css({position: 'fixed', left: '-250px', top: 'calc(50% - 138px)', transform: 'rotate(90deg)', transition: 'left 3s ease-in-out'}).show();
            $techbot.css({left: $body.outerWidth()});
        } else if (direction === 1) {
            $techbot.css({position: 'fixed', left: '-250px', top: '100%', transform: 'rotate(45deg)', transition: 'left 3s ease-in-out, top 3s ease-in-out'}).show();
            $techbot.css({left: $body.outerWidth(), top: '-250px'});
        } else {
            $techbot.css({position: 'fixed', left: 'calc(50% - 138px)', top: '100%', transition: 'top 3s ease-in-out'}).show();
            $techbot.css({top: '-250px'});
        }
    }*/
})();