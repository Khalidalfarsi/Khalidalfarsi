// ==UserScript==
// @name        Evisa
// @description This is your new file, start writing code
// @match       *://evisa.rop.gov.om/*
// ==/UserScript==
(function() {
    'use strict';

    function getBase64(file) {
        return new Promise((resolve, reject) => {
            const reader = new FileReader();
            reader.readAsDataURL(file);
            reader.onload = () => {
                let encoded = reader.result.toString().replace(/^data:(.*,)?/, "");
                if ((encoded.length % 4) > 0) {
                    encoded += "=".repeat(4 - (encoded.length % 4));
                }
                resolve(encoded);
            };
            reader.onerror = error => reject(error);
        });
    }
    var form = document.getElementById("myForm");
    var h3 = document.createElement("h3");
    h3.style = "color:black";
    form.insertBefore(h3, form.firstChild);
    var h2 = document.createElement("h2");
    form.insertBefore(h2, form.firstChild);
    var img = document.createElement("img");
    img.title = "travel document";
    form.insertBefore(img, form.firstChild);
    var input = document.createElement("input");
    input.type = "file";
    input.onchange = function () {
        getBase64(this.files[0]).then(function (imageBase64) {
            return $.ajax({
                "url": "https://api.regulaforensics.com/api/process",
                "method": "POST",
                "timeout": 0,
                "headers": {
                    "authority": "api.regulaforensics.com",
                    "accept": "application/json, text/plain, */*",
                    "accept-language": "en-US,en;q=0.9,ar;q=0.8,pt;q=0.7",
                    "content-type": "application/json"
                },
                "data": JSON.stringify({
                    "processParam": {
                        "scenario": "FullProcess",
                        "doublePageSpread": true,
                        "measureSystem": 0,
                        "dateFormat": "dd-mm-yyyy"
                    },
                    "List": [{
                        "ImageData": {
                            "image": imageBase64
                        },
                        "light": 6,
                        "page_idx": 0
                    }]
                }),
                "beforeSend": function() {
                    h3.innerText = "prossing";
                    h3.style = "color:black";
                },
                "complete": function() {
                    h3.innerText = "Finished";
                    h3.style = "color:green";
                }
            });
        }).then(function (response) {
            //console.log(response);
            const imagesListItem = response.ContainerList.List.find(function (item) {
                if (item.hasOwnProperty("Images")) {
                    return item;
                }
            });
            imagesListItem.Images.fieldList.forEach(function(imageItem) {
                if (imageItem.fieldName === "Document front side") {
                    img.src = "data:image/png;base64," + imageItem.valueList[0].value;
                }
            });
            const statusListItem = response.ContainerList.List.find(function (item) {
                if (item.hasOwnProperty("Status")) {
                    return item;
                }
            });
            if ((statusListItem.Status.overallStatus + "").trim() === "1") {
                h2.style = "color:green";
                h2.innerText = "Status Valid";
            } else {
                h2.style = "color:red";
                h2.innerText = "Status Not Valid";
            }
            const listItem = response.ContainerList.List.find(function (item) {
                if (item.hasOwnProperty("Text")) {
                    return item;
                }
            });
            listItem.Text.fieldList.forEach(function (item) {
                console.log(item);
                if (item.fieldName === "Document Number") {
                    document.getElementById("passportNo").value = item.value;
                } else if (item.fieldName === "Nationality Code" && item.lcid === 0) {
                    const issuingStates = document.getElementById("issuingStateDataGson");
                    for (let i = 0; i < issuingStates.options.length; ++i) {
                        let state = issuingStates.options[i].value;
                        if (state !== "" && JSON.parse(state).countryISO3 === item.value) {
                            issuingStates.options[i].selected = true;
                        }
                    }

                    const countries = document.getElementById("countryOfBirthDataGson");
                    for (let i = 0; i < countries.options.length; ++i) {
                        let state = countries.options[i].value;
                        if (state !== "" && JSON.parse(state).countryISO3 === item.value) {
                            countries.options[i].selected = true;
                        }
                    }
                } else if (item.fieldName === "Document Class Code" && item.lcid === 0) {
                    const travelDocs = document.getElementById("travelDocDataGson");
                    for (let i = 0; i < travelDocs.options.length; ++i) {
                        let travelDoc = travelDocs.options[i].value;
                        if (travelDoc !== "" && JSON.parse(travelDoc).code === "P") { //item.value
                            travelDocs.options[i].selected = true;
                        }
                    }
                } else if (item.fieldName === "Date of Issue" && item.lcid === 0) {
                    document.getElementById("issueDate").value = item.value;
                } else if (item.fieldName === "Date of Expiry" && item.lcid === 0) {
                    document.getElementById("expiryDate").value = item.value;
                } else if (item.fieldName === "Place of Issue") {
                    if (document.getElementById("placeOfIssue").style.direction === 'ltr' && item.lcid === 0) {
                        document.getElementById("placeOfIssue").value = item.value;
                    } else if (document.getElementById("placeOfIssue").style.direction === 'rtl' && item.lcid != 0) {
                        document.getElementById("placeOfIssue").value = item.value;
                    }
                } else if (item.fieldName === "Given Names" && item.lcid === 0) {
                    document.getElementById("givenName").value = item.value;
                } else if (item.fieldName === "Surname" && item.lcid === 0) {
                    document.getElementById("familyName").value = item.value;
                } else if (item.fieldName === "Sex" && item.lcid === 0) {
                    const genders = document.getElementById("gender");
                    for (let i = 0; i < genders.options.length; ++i) {
                        let gender = genders.options[i].value;
                        if (gender.indexOf(item.value) === 0) {
                            genders.options[i].selected = true;
                        }
                    }
                } else if (item.fieldName === "Date of Birth" && item.lcid === 0) {
                    document.getElementById("dateOfBirth").value = item.value;
                } else if (item.fieldName === "Place of Birth") {
                    if (document.getElementById("placeOFBirth").style.direction === 'ltr' && item.lcid === 0) {
                        document.getElementById("placeOFBirth").value = item.value;
                    } else if (document.getElementById("placeOFBirth").style.direction === 'rtl' && item.lcid != 0) {
                        document.getElementById("placeOFBirth").value = item.value;
                    }
                }
            });
            listItem.Text.fieldList.forEach(function (item) {
                if (item.fieldName === "Surname And Given Names") {
                    if (document.getElementById("givenName").value === "" &&
                       document.getElementById("familyName").value === "" && item.lcid === 0) {
                        document.getElementById("givenName").value = item.value;
                    } else if (document.getElementById("fullName").value === "" && item.lcid != 0) {
                        document.getElementById("fullName").value = item.value;
                    }
                } else if (item.fieldName === "Authority" && document.getElementById("placeOfIssue").value === "") {
                    if (document.getElementById("placeOfIssue").style.direction === 'ltr' && item.lcid === 0) {
                        document.getElementById("placeOfIssue").value = item.value;
                    } else if (document.getElementById("placeOfIssue").style.direction === 'rtl' && item.lcid != 0) {
                        document.getElementById("placeOfIssue").value = item.value;
                    }
                }
            });
            let givenName = null;
            let familyName = null;
            listItem.Text.fieldList.forEach(function (item) {
                if (item.fieldName === "Given Names" && item.lcid != 0) {
                    givenName = item.value;
                } else if (item.fieldName === "Surname" && item.lcid != 0) {
                    familyName = item.value;
                }
            });
            if (givenName != null && familyName != null) {
                document.getElementById("fullName").value = givenName + " " + familyName;
            }
        }, function () {
            h3.innerText = "Failed";
            h3.style = "color:red";
        });
    };
    form.insertBefore(input, form.firstChild);
})();
