/* global FB */
/* global window */
/* global google */
/* global getBaseUrl */
/* global externalLogOn */
/* global onAfterExternalLogOn */
/* global sendMessageOnServer */
/* global loadUserData */
/* global getQueryVariable */
/* eslint-disable no-console */
/* eslint-disable our-rules/white-label-feature */
/* eslint-disable our-rules/no-miss-parent-callback-in-async */
/* eslint-disable our-rules/no-miss-error-in-async-callback */

window.checkAuth = function (url, callback) {
    callback();
};

window.externalLogOn = function externalLogOn(response, callback) {
    var data = {
        id: response.id,
        email: response.email,
        name: response.name,
        portalID: response.portalID,
        token: response.token,
        credential: response.credential,
    };

    if (window.onAfterLogOnInFrame) {
        data.source = 'cms';
    }

    $.ajax({
        type: 'POST',
        url: getBaseUrl() + '/Account/ExternalLogOn',
        data: data,
        success: function (logonResponse) {
            if (!logonResponse.errorId) {
                if (typeof callback === 'function') {
                    response.isSignup = logonResponse.isSignup;
                    return callback(null, response);
                }
            } else if (logonResponse.errorId === 211 && logonResponse.url) {
                window.location.href = logonResponse.url;
            } else {
                return callback('Unable to login. Please login via Nicepage. Response error.', logonResponse);
            }
        },
        error: function (err) {
            return callback('Unable to login. Please login via Nicepage. Request error.', err);
        },
        dataType: 'json',
    });
};

/* Google Sign-in Start */

window.addEventListener('load', function () {
    if (typeof google === 'undefined') {
        return;
    }
    try {
        google.accounts.id.initialize({
            context: 'signin',
            client_id: window.serverSettings.googleAppId,
            callback: window.handleCredentialResponse,
            use_fedcm_for_prompt: true,
        });

        if (window.renderGoogleSignIdButton) {
            google.accounts.id.renderButton(
                document.getElementById('google-signin-button'),
                {
                    theme: 'filled_blue',
                    size: 'large',
                    text: window.renderSignUpGoogleButton ? 'signup_with' : 'signin_with',
                    width: '250',
                    locale: 'en',
                }
            );
        }

        // display the One Tap dialog
        if (!window.hideGoogleSignIdPromt) {
            google.accounts.id.prompt();
        }
    } catch (e) {
        console.error(e);
        if (typeof sendMessageOnServer === 'function') {
            sendMessageOnServer('auth.js google.accounts.id.initialize error', {error: JSON.stringify(e, Object.getOwnPropertyNames(e))});
        }
    }
});

window.handleCredentialResponse = function handleCredentialResponse(response) {
    var request = {
        portalID: 22,
        credential: response.credential,
    };

    externalLogOn(request, onAfterExternalLogOn);
};

/* Google Sign-in End */

/* Facebook Sign-in Start */

window.onFacebookLogin = function onFacebookLogin() {
    //eslint-disable-next-line no-negated-condition
    if (!window.fbAuthorized) {
        FB.login(function (response) {
            if (response.authResponse) {
                loadUserData(onAfterExternalLogOn);
            }
        },
        {scope: 'email'});
    } else {
        loadUserData(onAfterExternalLogOn);
    }
};

window.loadUserData = function loadUserData(callback) {
    FB.api('/me?fields=id,name,email', function (response) {
        if (!response.error) {
            if (window.isAuthenticated) {
                return callback(null, response);
            }
            window.fbAuthorized = true;
            // login or register user
            var authResponse = FB.getAuthResponse();
            response.portalID = 21;
            if (authResponse) {
                response.token = authResponse.accessToken;
            }

            externalLogOn(response, callback);
            return;
        }

        window.fbAuthorized = false;
        callback(response.error);
    });
};

/*Facebook Sign-in End */

window.onAfterExternalLogOn = function onAfterExternalLogOn(error, response) {
    if (error) {
        console.error(error);
        if (typeof sendMessageOnServer === 'function') {
            sendMessageOnServer('auth.js onAfterExternalLogOn error.',
                {
                    errorMsg: JSON.stringify(error, Object.getOwnPropertyNames(error)),
                    error: JSON.stringify(response, Object.getOwnPropertyNames(response)),
                });
        }
        return;
    }

    if (window.onAfterLogOnInFrame) {
        window.location = '/Editor/frame/Account/ProfileData?formaction=' + (response.isSignup ? 'registration' : 'login');
    } else {
        var baseUrl = getBaseUrl();
        var returnUrl = getQueryVariable('returnurl');
        if (returnUrl && returnUrl.length > 1 &&
            returnUrl[0] === '/' &&
            returnUrl[1] !== '/') {
            window.location = baseUrl + returnUrl;
        } else if (response.isSignup) {
            window.location = baseUrl + '/download';
        } else if (!window.location.pathname.toLowerCase().startsWith('/editor/siteboard')) {
            window.location = baseUrl + '/';
        }
    }
};

window.getQueryVariable = function getQueryVariable(variable) {
    var query = window.location.search.substring(1);
    var vars = query.split('&');
    for (var i = 0; i < vars.length; i++) {
        var pair = vars[i].split('=');
        if (pair[0].toLowerCase() === variable) {
            return decodeURIComponent(pair[1]);
        }
    }
    return false;
};
